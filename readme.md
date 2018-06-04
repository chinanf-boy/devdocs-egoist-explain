# devdocs-desktop [![explain](http://llever.com/explain.svg)](https://github.com/chinanf-boy/Source-Explain)

「 Desktop client for devdocs.io 」

应该第一个 `electron 应用` explain, 所以选了个相对程度但不复杂的

Explanation

> "version": "1.0.0"

[github source](https://github.com/egoist/devdocs-desktop)

[中文](./readme.md) | ~~[english](./readme.en.md)~~

---

<span> 穷 </span>

<a href="https://patreon.com/yobrave">
<img src="https://c5.patreon.com/external/logo/become_a_patron_button@2x.png" height="30"></a> 
---

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [package.json](#packagejson)
  - [1.  `postinstall`](#1--postinstall)
  - [2.  electron-builder](#2--electron-builder)
  - [3.  `build`](#3--build)
- [app/index.js](#appindexjs)
  - [electron-控制-方式](#electron-%E6%8E%A7%E5%88%B6-%E6%96%B9%E5%BC%8F)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

---

## package.json

``` js
  "main": "app/index.js", // 主入口
  "scripts": {
    "postinstall": "electron-builder install-app-deps",
    "test": "npm run lint",
    "lint": "xo",
    "app": "DEBUG=devdocs-desktop:* electron app/index.js",
    "pack": "build --dir",
    "dist": "build",
    "release": "build"
  },
```

### 1.  `postinstall`

在安装软件包后运行。 

> `electron-builder install-app-deps` 下载 应用的 依赖

### 2.  [electron-builder](https://github.com/electron-userland/electron-builder)

一个完整的解决方案，打包并构建一个准备发布的电子应用程序，带有“自动更新”支持

### 3.  `build`

提供 使用 `elertron-builder` 的快捷, 同时依赖`package.json`-`build`字段的定义

<details>

<summary> 本应用 build 字段 细节 </summary>

``` js
 "build": {
    "appId": "com.egoistian.devdocs", // id
    "productName": "DevDocs", // name
    "compression": "maximum", // 压缩级别
    "asar": true, // 是否使用 Electro n的存档格式将应用程序的源代码打包到档案中。
    "mac": {
      "category": "public.app-category.developer-tools" // 应用程序类别类型
    },
    "win": {
      "target": [ // 目标封装类型
        "nsis", // Windows的默认目标
        "zip",
        "portable" // 便携式应用，而无需安装
      ]
    },
    "nsis": {
      "oneClick": false // 是创建一键安装还是辅助
    },
    "linux": {
      "synopsis": "DevDocs desktop app", // 简短说明
      "category": "Development", // 应用程序类别
      "target": [ // 目标封装类型
        "AppImage", // 让Linux应用随处运行 
        "deb", // Debian软件包选项
        "tar.xz"
      ]
    }
  }
```

- [build 全配置](https://www.electron.build/configuration/configuration) `en`
- [AppImage](https://appimage.org/)

</details>

## app/index.js

### electron-控制-方式

既然是 `electron` 应用

就有必要说明一下, [electron 控制网页 的 方式](./electron.md)

### require

``` js
const path = require('path')
const { app, BrowserWindow, Menu } = require('electron')
const debug = require('debug')('devdocs-desktop:index')
const createMenu = require('./menu')
const config = require('./config')
const tray = require('./tray')
const updater = require('./updater')
const { toggleGlobalShortcut } = require('./utils')
const login = require('./login')

require('electron-debug')() // 专用 调试信息
require('electron-context-menu')({ // 菜单
  showInspectElement: true // 能右键 点出 dev调试器
})

```

- [config](./config.md)


### 应用信息

``` js
//改变当前应用
app.setAppUserModelId('com.egoistian.devdocs')

let mainWindow
let isQuitting = false
let urlToOpen

```

### 唯一应用

``` js
// https://electronjs.org/docs/api/app#appmakesingleinstancecallback
const isAlreadyRunning = app.makeSingleInstance(() => {
  // 这将确保只有一个应用程序的实例正在运行, 其余的实例全部会被终止并退出。
  if (mainWindow) {
    if (mainWindow.isMinimized()) {
      mainWindow.restore()
    }

    mainWindow.show()
  }
})

if (isAlreadyRunning) {
  // 已经有了 , 退出
  app.quit()
}

```

### 关闭和显示

``` js
function toggleWindow() { // 关闭和显示 主窗口
  if (mainWindow.isVisible()) {
    mainWindow.hide()
  } else {
    mainWindow.show()
  }
}

```

### 主窗口

``` js
function createMainWindow() { // 创建主窗口
  const lastWindowState = config.get('lastWindowState')

  const win = new BrowserWindow({
    title: app.getName(),
    x: lastWindowState.x, // 开启的位置
    y: lastWindowState.y,
    width: lastWindowState.width, // 窗口大小
    height: lastWindowState.height,
    minWidth: 600,
    minHeight: 400,
    show: false, // 最初的可见性状态将为visible 尽管窗口实际上是隐藏的。
    titleBarStyle: 'hidden', // 窗口标题栏的样式
    backgroundColor: '#ffffff'
  })

  if (process.platform === 'darwin') { // 如果是 macOS 系统
    win.setSheetOffset(24)
  }

  const url = `file://${path.join(__dirname, 'renderer', 'index.html')}`

  win.loadURL(url) // 加载 web页面

  win.on('close', e => {
    if (!isQuitting) {
      e.preventDefault()

      if (process.platform === 'darwin') {
        // 一般 在 macos 中 需要 Command + Q 才能完全退出程序
        app.hide()
      } else {
        win.hide()
      }
    }
  })

  return win
}

```

- [BrowserWindow 参数](https://electronjs.org/docs/api/browser-window#new-browserwindowoptions)

---

### login

登录信息

``` js
app.on('login', (event, webContents, request, authInfo, cb) => {
  debug('app.on(login)')
  event.preventDefault()
  login(cb)
})

```

- [login](./login.md)


---

### ready

应用准备好就, 开启

``` js
app.on('ready', () => {
  
  // 快捷键
  const shortcut = config.get('shortcut')
  for (const name in shortcut) {
    const accelerator = shortcut[name]
    if (accelerator) {
      toggleGlobalShortcut({
        name,
        accelerator,
        registered: false,
        action: toggleWindow
      })
    }
  }

    // 菜单
  Menu.setApplicationMenu(
    createMenu({
      toggleWindow
    })
  )
  // 主窗口
  mainWindow = createMainWindow()
  // 将图标和上下文菜单添加到系统的通知区域。
  tray.create(mainWindow)

// 加载页面时，ready-to-show如果窗口尚未显示，则渲染器进程首次渲染页面时会发出事件
  mainWindow.once('ready-to-show', () => {
    mainWindow.show()
    updater.init() // 更新初始化
    if (urlToOpen) {
      mainWindow.webContents.send('link', urlToOpen)
    }
  })
})

```

- [x] [toggleGlobalShortcut](./util.md)

添加或删除全局快捷键

- [createMenu](./menu.md)

应用菜单列表

- [x] [updater](./updater.md)

自动更新触发

- [x] [tray](./tray.md)

系统菜单程序列表的图标

- [ready-to-show](https://electronjs.org/docs/api/browser-window#using-ready-to-show-event)

- [webContents.send](https://electronjs.org/docs/api/web-contents#contentssendchannel-arg1-arg2-)

除了`ipcMain` 之外, 为了有目的性的发送, 窗口实例也是具有向 `页面进程` 发送信息的能力

### 激活

``` js
// 应用程序激活时
app.on('activate', () => {
  mainWindow.show()
})

```

### 获得焦点

``` js
let hasOpenedOnce
// 在 browserWindow 获得焦点时发出。
app.on('browser-window-focus', () => {
  if (hasOpenedOnce) {
    mainWindow.webContents.send('focus-webview')
  } else {
    hasOpenedOnce = true
  }
})

```

### 退出前

``` js
// 退出前
app.on('before-quit', () => {
  isQuitting = true

  if (!mainWindow.isFullScreen()) { // 只要不是全屏,就保存位置
   // 因为是 elertron-store 库, 所以其实是保存下来的
    config.set('lastWindowState', mainWindow.getBounds())
  }
})

```

### 自定义协议

``` js
// devdocs://
// 应用内部都是已 这个链接开头
app.setAsDefaultProtocolClient('devdocs')
```

### 基本启动

应用程序完成基本启动时发出

``` js
app.on('will-finish-launching', () => {
  //当用户想要在应用中打开一个 URL 时发出
  app.on('open-url', (e, url) => {
    // 1. 第一次 应用还没有准备好的时候, 窗口没有生成
    if (mainWindow) {
      // 3. 有了, 就请求
      mainWindow.webContents.send('link', url)
    } else {
      // 2. 先保存
      urlToOpen = url
    }
  })
})

```

> 您通常会在此处设置侦听器`open-file`和 `open-url`事件

- [will-finish-launching](https://electronjs.org/docs/api/app#event-will-finish-launching)

- [open-url](https://electronjs.org/docs/api/app#%E4%BA%8B%E4%BB%B6-open-url-macos)

- [webContents.send](https://electronjs.org/docs/api/web-contents#contentssendchannel-arg1-arg2-)

除了`ipcMain` 之外, 为了有目的性的发送, 窗口实例也是具有向 `页面进程` 发送信息的能力
