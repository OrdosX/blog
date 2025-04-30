---
title: 用于B站首页净化与增强的油猴脚本
date: 2024-11-26 20:08:47
categories:
- 小工具
tags:
- JavaScript
- Tampermonkey
---
这个脚本通过多种方式优化B站首页的使用体验，解决了首页推荐内容杂乱、视频预览效率低、点击链接泄露隐私、垃圾关键词充斥搜索框等痛点。主要功能包括推荐卡片净化、视频预览加速、预览进度继承、无痕复制链接、搜索内容净化，帮助用户定制首页内容，提升观看与使用体验。

<!--more-->

## 1. 推荐卡片净化

首页卡片中有很多耍猴的引流广告、低俗的擦边直播，以及点了无数次“不感兴趣”也屏蔽不掉的低创视频，还有分区卡片占用了整整一栏，严重影响体验。

此脚本可以清理首页上不必要的推送内容，包括广告、直播和分区推荐卡片等，并且可以按UP主名屏蔽低质量视频，确保只看到自己感兴趣的内容。

### 实现方法

垃圾卡片会有额外的CSS类，例如直播卡片的类为`bili-live-card`，通过检查节点的`classList`是否包含对应的类，即可屏蔽掉直播、楼层、广告卡片。

如果同时使用广告屏蔽插件，如uBlock Origin，广告卡片会被屏蔽但留下一个写着“内容被屏蔽”的空白卡片，也很烦人，因此脚本也会删除没有链接的空卡片。

脚本使用正则表达式匹配一些常见的低创UP主ID格式，可以根据需求增删。

代码逻辑如下：

```javascript
const BLOCKED_AUTHORS_REGEX = [
    /剪辑/,
    /话题/,
    /记录/,
    /纪录/,
    /兔警长/,
    /混合体主脑/,
    /BB姬/,
    /档案/
];

function remove_unwanted_card(node) {
    if (!node.classList) return;
    // 首先检查是否是特定类型的卡片，比如直播卡片、广告卡片等
    if (
        node.classList.contains('bili-live-card') ||
        node.classList.contains('floor-single-card') ||
        (node.classList.contains('bili-video-card') && node.querySelector('a')?.href.startsWith('https://cm')) ||
        (node.classList.contains('bili-video-card') && !node.querySelector('a'))
    ) {
        node.remove();
    }

    // 其次检查UP主是否属于屏蔽列表中的关键词
    const authorNode = node.querySelector('.bili-video-card__info--author');
    if (authorNode && BLOCKED_AUTHORS_REGEX.some(regex => regex.test(authorNode.innerText))) {
        node.remove();
    }
}
```

## 2. 倍速悬停预览

悬停在卡片上预览视频时，默认按原速播放，往往需要浪费大量时间去看不确定是否有趣的内容。

这个脚本在用户悬停视频卡片时，以三倍速播放视频，帮助用户快速浏览视频，从而判断视频是否值得观看。

### 实现方法

脚本使用了 `PLAYBACK_RATE` 常量来设定悬停时的播放倍速，具体的实现步骤如下：

- 首先，通过 `loadedmetadata` 事件来确保视频的元数据已加载完毕，从而能够安全地设置播放速度。
- 使用 `node.ontimeupdate` 事件监听播放进度，并将预览的时间记录下来，以便后续可以继承。

代码如下：

```javascript
const PLAYBACK_RATE = 3;

function set_faster_preview(node) {
    if (!(node instanceof HTMLVideoElement)) return;

    node.addEventListener('loadedmetadata', () => {
        node.playbackRate = PLAYBACK_RATE;
        const parent_anchor = node.closest('a');
        parent_anchor.dataset.previewTime = node.currentTime;
        parent_anchor.dataset.previewDuration = node.duration;
        node.ontimeupdate = () => {
            const anchor_dataset = parent_anchor.dataset;
            if (anchor_dataset.previewTime < node.currentTime) {
                anchor_dataset.previewTime = node.currentTime;
            }
        };
    });
}
```

通过这种方式，用户在悬停时可以以三倍速度预览视频内容，从而高效决策是否点击进入观看。

## 3. 无痕复制链接

有时用户想吃瓜而不想污染信息流，往往会复制链接到无痕浏览模式中观看，但右键复制时链接会被自动加上跟踪字符串，行为还会被上传，和直接点进去没什么区别。这个脚本通过为每个视频卡片提供“一键复制”功能，实现无痕复制链接，避免触发任何点击事件，从而保护用户隐私。

### 实现方法

脚本通过为每个视频卡片动态添加一个按钮来实现无痕复制链接的功能，具体步骤如下：

- 首先，通过选择器定位到卡片信息的底部容器。
- 创建一个新按钮，设置文本为“复制链接”，并将其添加到视频卡片的底部。
- 为按钮绑定点击事件，点击后使用 `GM_setClipboard` 将视频链接复制到剪贴板，同时显示“已复制”的提示。

代码如下：

```javascript
function add_copy_link_button(node) {
    if (!(node.classList && node.classList.contains('bili-video-card'))) return;
    const bottomContainer = node.querySelector('.bili-video-card__info--bottom');
    if (!bottomContainer) return;

    const copyButton = document.createElement('a');
    copyButton.textContent = '复制链接';
    copyButton.style.position = 'absolute';
    copyButton.style.display = 'flex';
    copyButton.style.right = '0';
    copyButton.href = '#';
    bottomContainer.appendChild(copyButton);

    copyButton.addEventListener('click', (event) => {
        event.preventDefault();
        const videoLinkElement = bottomContainer.previousElementSibling.querySelector('a');
        if (videoLinkElement) {
            const videoUrl = videoLinkElement.href;
            GM_setClipboard(videoUrl);
            copyButton.textContent = '已复制';
            setTimeout(() => {
                copyButton.textContent = '复制链接';
            }, 1000);
        }
    });
}
```

这个功能让分享视频变得非常简单，用户无需手动查找和复制链接，从而大幅度提高了操作的便利性。

## 4. 搜索净化

搜索框的默认搜索词常出现一些耸人听闻的词，甚至是谣言，点进去搜又没有任何有用的内容。这个脚本删除搜索框中的默认内容和热搜信息，让用户在搜索时能够专注于真正有价值的内容，提升搜索体验。

### 实现方法

该功能分为两部分：清空搜索框的默认内容和移除热搜。

- 清空搜索框默认内容：通过移除搜索框的 `placeholder` 和 `title` 属性，确保用户可以在干净的输入框中进行搜索。
- 移除热搜内容：通过 `MutationObserver` 观察搜索面板的节点变化，一旦检测到包含热搜的元素，立即将其删除。

代码如下：

```javascript
function purify_search() {
    const searchBox = document.querySelector('.nav-search-input');
    if (searchBox) {
        searchBox.attributes.removeNamedItem('placeholder');
        searchBox.attributes.removeNamedItem('title');
    }

    (new MutationObserver((mutationsList) => {
        mutationsList.forEach((mutation) => {
            mutation.addedNodes.forEach((node) => {
                if (node.nodeType === Node.ELEMENT_NODE) {
                    const trendingElements = node.classList.contains('trending')
                        ? [node]
                        : node.querySelectorAll('.trending');
                    trendingElements.forEach(trending => trending.remove());
                }
            });
        });
    })).observe(document.querySelector('.search-panel'), {
        childList: true,
        subtree: true
    });
}
```

通过这一功能，用户在搜索时可以免受默认内容和热门推荐的干扰，更专注于自己的搜索目标。

## 5. 预览进度继承

预览视频后，如果感兴趣进入正式观看，还需要手动跳过已经预览过的进度，非常麻烦。这个脚本在正式播放视频时，自动继承预览进度，让用户无需手动跳过已预览的部分，提升观看体验。

### 实现方法

在用户点击视频链接时，脚本会检查用户的预览进度，并将这一信息添加到视频链接中。具体步骤如下：

- 首先，检查数据集中的预览时间和视频时长，确保有有效的预览时间。
- 然后，将预览时间作为参数添加到链接中，使用户进入视频页面时直接从该时间点开始播放。

代码如下：

```javascript
function inherit_preview_progress(node, url) {
    const preview_time = Number(node.dataset.previewTime);
    const preview_duration = Number(node.dataset.previewDuration);
    if (!preview_time || preview_time < 5 * PLAYBACK_RATE || (preview_duration < 299 && preview_time + 3 > preview_duration)) return;

    url.searchParams.append('t', Math.floor(preview_time));
    node.href = url.toString();
}

window.addEventListener('click', e => {
    const anchor_element = e.target.closest('a');
    if (!anchor_element || anchor_element.href.length === 0) return;
    const url_object = new URL(anchor_element.href);

    inherit_preview_progress(anchor_element, url_object);
});
```

这个功能通过记录并继承预览进度，让用户的观看过程更加流畅和个性化，避免了重复的内容，从而提升了整体观看体验。

