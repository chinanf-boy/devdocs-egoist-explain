## electron

---

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [简要说明](#%E7%AE%80%E8%A6%81%E8%AF%B4%E6%98%8E)
- [页面进程](#%E9%A1%B5%E9%9D%A2%E8%BF%9B%E7%A8%8B)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

---

### 简要说明

1. electron 有个`主进程`

``` js
const { app, BrowserWindow } = require('electron')
```

2. 所有你看到的`页面进程「渲染进程」`都是通过 `主进程` 生产的

``` js
new BrowserWindow
```

3. 页面进程之间, 不能直接通信, 需要借助 主进程`ipc`

- 主进程使用`ipcMain` - [ipcMain doc](https://electronjs.org/docs/api/ipc-main#%E5%8F%91%E9%80%81%E6%B6%88%E6%81%AF)

- 页面进程使用`ipcRenderer` - [ipcRenderer doc](https://electronjs.org/docs/api/ipc-renderer#ipcrenderer)


> 使用方式就是 监听`on("name",function)`, 发送`send("name",value)`


### 页面进程

1. 如需请求本机代码「`js`」, 需要通过 electron 库中的 `remote`

``` html
<!-- 页面进程 *.html -->
<script>
const js = require('electron').remote.require("./local.js")
</script>
```

> 当然 `local.js` 也要导出才行

``` js
// local.js
exports {something}
```

2. 对于 `node` 中有关系统控制, 就看 `electron` 支持的程度了

``` js
const fs = require('fs')
const path = require('path')
```

在 `页面进程 html`

``` html
<script>
const fs = require('fs')
const path = require('path')
// 当然 同一目录下 
const m = require('./main')
// 可以使用
</script>
```

3. 从 `npm` 安装的 `node_modules`

未实践,待续