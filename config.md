## config


每次操作都是一次 文件级别的 存储, 应用退出也不会变


``` js
const Config = require('electron-store')

module.exports = new Config({
  defaults: {
    lastWindowState: { // 上一次 窗口 位置
      width: 800,
      height: 600
    },
    shortcut: { // 快捷键
      toggleApp: null
    },
    mode: 'dark' // 颜色主题
  }
})

```

- https://github.com/sindresorhus/electron-store