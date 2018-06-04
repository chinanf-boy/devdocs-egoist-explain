## util

---

### configDir

主目录上文件的路径

``` js
const os = require('os')
const path = require('path')
const home = os.homedir()

exports.configDir = function(...args) {
return path.join(home, '.devdocs', ...args)
}
```

### toggleGlobalShortcut

添加与删除快捷键

> `app/index.js ` 使用

``` js
    if (accelerator) {
      toggleGlobalShortcut({
        name,
        accelerator,
        registered: false, // 一定是 false
        action: toggleWindow
      })
```

``` js
const { globalShortcut } = require('electron')
const config = require('./config') // 会保存下来的存储

exports.toggleGlobalShortcut = function({
  name,
  registered,
  accelerator,
  action
}) {
  if (registered) { // 已注册的 删除
  // 但在 app/index.js 中是不会删除
    globalShortcut.unregister(accelerator)
    config.delete(`shortcut.${name}`) // 存储删除
  } else {
    // 没注册的 添加
    const ret = globalShortcut.register(accelerator, action)
    config.set(`shortcut.${name}`, accelerator) // 存储
    if (!ret) {
      console.error(`Failed to register ${accelerator}`)
    }
  }
}

```

- [globalShortcut](https://electronjs.org/docs/api/global-shortcut#globalshortcut)

globalShortcut 模块可以在操作系统中注册/注销全局快捷键, 以便可以为各种快捷方式自定义操作。