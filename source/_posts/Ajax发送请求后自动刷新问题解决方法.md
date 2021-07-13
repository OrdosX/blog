---
title: Ajax发送请求后自动刷新问题解决方法
date: 2018-11-12 15:17:44
categories:
- 前端
tags:
- ajax
---
进行Ajax操作时，有时会遇到在发送完请求后页面自动刷新的情况，导致结果无法显示。我查阅资料找到了这个问题的解决方法，记录在此以供参考。

<!--more-->

## 问题

使用如下代码进行Ajax操作，发送完请求后页面都会被刷新，但是抓包的结果中返回值正常。

```html
<script>
    $(document).ready(function () {
        $("button").click(function () {
            $.post("/handlers/login", JSON.stringify(GetJsonData()),
                function (data, status) {
                    alert("Data: " + data + "\nStatus: " + status);
                });
        });
    });
    function GetJsonData() {
        var json = {
            "username": $("#inputUsername").val(),
            "password": $.base64.btoa($("#inputPassword").val())
        };
        return json;
    }
</script>

<body>
    <form>
        <h2>Please sign in</h2>
        <input type="text" id="inputUsername">
        <input type="password" id="inputPassword">
        <button>发送</button>
    </form>
</body>
```

## 解决

经查，问题出现在`<button>发送</button>`一行。按钮执行操作后会自动刷新页面。

所以，只要将其改为`<input type="button" id="button">`，然后第三行的`$("button").click()`改为`$("#button").click()`即可。