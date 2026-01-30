---
title: 用 Rust 和 Tauri 2 实现动态托盘电量图标
date: 2025-05-30 13:40:08
categories:
- 桌面应用
tags:
- Rust
- Tauri
---

在设计电池电量实时显示的程序 [percentage-rust](https://github.com/OrdosX/percentage-rust) 时，需要在 Tauri 2 中，将动态的文本显示到系统托盘图标上。网上没有现成的资料，于是把自己实现的方法分享出来，以供参考。
<!-- more -->

最近打算学 Rust，想找个项目做牵引。这个项目灵感来源于 [Percentage](https://github.com/kas/percentage)，它是一个基于 C++的、在 Windows 的系统托盘中显示电量的程序。系统自带的电池图标只能看个大概，若要查看具体电量还需要把鼠标悬停在图标上。更要命的是，如果因电量不足开启了省电模式，那个叶子图标会把剩余的电量挡住，完全不知道还剩下多少。Percentage 则通过把电池电量百分比以数字形式实时显示在任务栏中，解决了这个痛点，让人瞟一眼就知道当前剩余电量。
编写过程中，为了实现“将获取到的电池状态绘制为图标、更新到系统托盘”这个功能，搜索了许多资料。然而，Tauri 2 的社区内容尚不是特别完善，官方教程的示例也只是提到了使用静态 icon 文件作为托盘图标的方法，说白了没地方直接抄，于是自己研究并实现了这个功能。
思路为：先生成字符串，然后调整字体大小使文字不超出图标宽度，接着计算坐标使文字居中，再渲染得到一个 `tauri::image::Image` 对象，最后将这个对象通过 `tauri::tray::TrayIcon()` 函数更新为当前图标。
核心踩坑点有两个。其一是要使用 `ascent` 而不是 `height` 作为字符高度，否则数字位置会偏高。其二是要将图像编码为 ICO 格式存储到二进制 buffer 中，再用 buffer 初始化 `tauri::image::Image`，而不是直接使用 `image::RgbaImage`，否则 Tauri 不认。
## 字体参数计算

首先，使用 `ab_glyph` 计算文本尺寸，参考其 [文档](https://docs.rs/ab_glyph/0.2.29/ab_glyph/trait.Font.html#glyph-layout-concepts) 可知，字体的几个主要尺寸如下所示：
```
         ‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾
                  |  .:x++++==              |
                  | .#+                     |
                  | :@            =++=++x=: |
           ascent | +#       x:  +x     x+  |
                  | =#       #:  :#:---:#:  | height
                  | -@-      #:  .#--:--    |
                  |  =#:-.-==#:   #x+===:.  |
baseline ____________ .-::-. ..  #:    .:@. |
                  |              #+--..-=#. |
          descent |               -::=::-   |
         ____________________________________
                | |             |           | line_gap
                | |  h_advance  |           ‾
                 ^                      
           h_side_bearing
```

其中，宽度是 `h_advance` 很显然。但对于高度，由于程序中只考虑数字和几个符号，如果使用 `height` 会导致字符位置偏上，所以应该使用 `ascent`。最终实现如下：
```rust
fn measure_text(&self, text: &str, scale: PxScale) -> (f32, f32) {
    let scaled_font = self.font.as_scaled(scale);
    let width = text.chars().map(|c| scaled_font.h_advance(scaled_font.glyph_id(c))).sum();
    let height = scaled_font.ascent();
    (width, height)
}
```

函数中的 `scale` 是通过二分法确定的，用二分法尝试不同的 `scale`，逐步逼近（但不超过）画布的宽度。这样做的原因是，字体中不同字符的宽度略有区别，如果写死宽度，当显示较窄的字符时，会显得图标过于空旷。
确定最终的 `scale` 之后，再调用 `measure_text()` 得到宽高，与画布尺寸做运算，即可得到要使字符串居中所需的 x、y 坐标。
## 图标绘制与更新

在绘制图标的函数中，首先创建一个 `RgbaImage` 对象，用之前计算得到的 x, y, scale 绘制到画布，再将画布编码为 ICO 格式的二进制 buffer，用这个二进制 buffer 初始化 `tauri::image::Image` 对象，作为返回值。
```rust
fn render_icon(&self, x: i32, y: i32, scale: PxScale, text: &str) -> Result<Image<'static>> {
    let mut img = RgbaImage::new(BatteryIconGenerator::SIZE, BatteryIconGenerator::SIZE);
    draw_text_mut(&mut img, Rgba([0, 0, 0, 255]), x, y, scale, &self.font, &text);

    let mut icon_data = Cursor::new(Vec::new());
    img.write_to(&mut icon_data, image::ImageFormat::Ico)
        .context("Failed to encode icon to ICO")?;

    let icon_image = Image::from_bytes(&icon_data.into_inner())
        .context("Failed to create Tauri image")?
        .to_owned();

    Ok(icon_image)
}
```

主程序中，生成一个线程，每隔一秒调用 `battery` 库获取电池充电状态和电量百分比。
```rust
fn spawn_battery_monitor(tx: Sender<(u32, State)>) {
    thread::spawn(move || {
        let manager = Manager::new().expect("Failed to initialize battery manager");
        let mut last_battery_info: Option<(u32, State)> = None;

        loop {
            if let Ok(batteries) = manager.batteries() {
                if let Some(battery) = batteries.flatten().next() {
                    let percentage = (battery.state_of_charge().value * 100.0).round() as u32;
                    let state = battery.state();

                    if Some((percentage, state)) != last_battery_info {
                        last_battery_info = Some((percentage, state));

                        tx.blocking_send((percentage, state))
                            .expect("Failed to send battery info");
                    }
                }
            }
            thread::sleep(Duration::from_secs(1));
        }
    });
}
```

它绑定一组 rx、tx，其中 rx 用于初始化一个异步函数。
```rust
let (tx, rx) = channel(1);
spawn_battery_monitor(tx);
spawn_tray_updater(tray, rx);
```

当电池状态更新，rx 收到消息，就生成 icon 并通过 `set_icon()` 函数更新图标，实现实时电量显示。
```rust
fn spawn_tray_updater(tray: Arc<Mutex<TrayIcon>>, mut rx: Receiver<(u32, State)>) {
    async_runtime::spawn(async move {
        let icon_generator = BatteryIconGenerator::new().unwrap();
        while let Some((percentage, state)) = rx.recv().await {
            if let Ok(icon) = icon_generator.generate_icon(percentage, state == State::Charging).await {
                let tooltip = match state {
                    State::Charging => format!("Charging: {}%", percentage),
                    State::Discharging => format!("Discharging: {}%", percentage),
                    State::Full => format!("Full"),
                    _ => format!("Unhandled state: {}%", percentage),
                };

                let tray = tray.lock().await;
                if let Err(e) = tray.set_icon(Some(icon)) {
                    eprintln!("Failed to update tray icon: {}", e);
                }
                if let Err(e) = tray.set_tooltip(Some(&tooltip)) {
                    eprintln!("Failed to update tray tooltip: {}", e);
                }
            }
        }
    });
}
```