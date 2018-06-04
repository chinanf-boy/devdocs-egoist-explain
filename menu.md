## menu

许多的菜单选项定义

除了, 触发函数, 每个的配置其实差不多

<!-- START doctoc -->
<!-- END doctoc -->

---

``` js
const {
  Menu,
  shell,
  globalShortcut,
  BrowserWindow,
  dialog
} = require('electron')
const axios = require('axios')
const semverCompare = require('semver-compare')
const { configDir, toggleGlobalShortcut } = require('./utils')
const config = require('./config')
const pkg = require('./package')

function sendAction(action, ...args) { // 每个动作 给 窗口发送 信息
  const [win] = BrowserWindow.getAllWindows()

  if (process.platform === 'darwin') {
    // 将窗口从最小化状态恢复到以前的状态。
    win.restore() 
  }

  win.webContents.send(action, ...args)
}

function updateMenu(opts) {
  // 设置应用菜单
  Menu.setApplicationMenu(createMenu(opts))
}

```

### 创建菜单

``` js

function createMenu(opts) {
  const toggleAppAccelerator = config.get('shortcut.toggleApp') || 'CmdOrCtrl+Shift+D'
  const toggleAppAcceleratorRegistered = globalShortcut.isRegistered(
    toggleAppAccelerator
  )

```

- [在主进程中创建程序菜单的简单API模版示例](https://electronjs.org/docs/api/menu#%E7%A4%BA%E4%BE%8B)

### 偏爱

``` js
  const preferences = [
    {
      label: 'Preferences', // 一个菜单选项名称
      submenu: [ // 下拉列表
        {
          label: 'Custom CSS',
          click() {
            shell.openItem(configDir('custom.css'))
          }
        },
        {
          label: 'Custom JS',
          click() {
            shell.openItem(configDir('custom.js'))
          }
        },
        { // 是否使用 全局快捷键
          label: `${
            toggleAppAcceleratorRegistered ? 'Disable' : 'Enable'
          } Global Shortcut`,
          click() {
            toggleGlobalShortcut({
              name: 'toggleApp',
              registered: toggleAppAcceleratorRegistered,
              accelerator: toggleAppAccelerator,
              action: opts.toggleWindow
            })
            updateMenu(opts)
          }
        }
      ]
    },
    {
      type: 'separator'
    }
  ]

```

###  checkForUpdates

检查更新

``` js
  const checkForUpdates = {
    label: 'Check for Updates',
    async click(item, focusedWindow) {
      if (!focusedWindow) return

      const api =
        'https://api.github.com/repos/egoist/devdocs-desktop/releases/latest'
      const latest = await axios.get(api).then(res => res.data)

      if (semverCompare(latest.tag_name.slice(1), pkg.version) === 1) { // 比较版本
        dialog.showMessageBox(
          focusedWindow,
          {
            type: 'info',
            message: 'New updates!',
            detail: `A new release (${
              latest.tag_name
            }) is available, view more details?`,
            buttons: ['OK', 'Cancel'],
            defaultId: 0
          },
          selected => { // 默认 ok
            if (selected === 0) {
              // 在用户的默认浏览器中打开 URL 的示例:
              shell.openExternal(
                'https://github.com/egoist/devdocs-desktop/releases/latest'
              )
            }
          }
        )
      } else { // 没有更新
        dialog.showMessageBox(focusedWindow, {
          message: 'No updates!',
          detail: `v${pkg.version} is already the latest version.`
        })
      }
    }
  }

```

### 编辑

若要将菜单项的操作设置为标准操作, 应设置菜单项的 `role` 属性

``` js
  const template = [
    {
      label: 'Edit',
      submenu: [
        {
          role: 'undo'
        },
        {
          role: 'redo'
        },
        {
          type: 'separator'
        },
        {
          role: 'cut'
        },
        {
          role: 'copy'
        },
        {
          role: 'paste'
        },
        {
          role: 'pasteandmatchstyle'
        },
        {
          role: 'delete'
        },
        {
          role: 'selectall'
        }
      ]
    },

```

### 视图

``` js
    {
      label: 'View',
      submenu: [
        {
          label: 'Reset Text Size',
          accelerator: 'CmdOrCtrl+0',
          click() {
            sendAction('zoom-reset')
          }
        },
        {
          label: 'Increase Text Size',
          accelerator: 'CmdOrCtrl+Plus',
          click() {
            sendAction('zoom-in')
          }
        },
        {
          label: 'Decrease Text Size',
          accelerator: 'CmdOrCtrl+-',
          click() {
            sendAction('zoom-out')
          }
        },
        {
          type: 'separator'
        },
        {
          label: 'Search In Page',
          accelerator: 'CmdOrCtrl+F',
          click(item, focusedWindow) {
            focusedWindow.webContents.send('open-search')
          }
        },
        {
          type: 'separator'
        },
        {
          label: 'Reload',
          accelerator: 'CmdOrCtrl+R',
          click(item, focusedWindow) {
            if (focusedWindow) focusedWindow.reload()
          }
        },
        {
          label: 'Toggle Developer Tools',
          accelerator:
            process.platform === 'darwin' ? 'Alt+Command+I' : 'Ctrl+Shift+I',
          click(item, focusedWindow) {
            if (focusedWindow) focusedWindow.webContents.toggleDevTools()
          }
        }
      ]
    },

```

### 窗口

``` js
    {
      role: 'window',
      submenu: [
        {
          role: 'minimize'
        },
        {
          role: 'close'
        }
      ]
    }
  ]

  if (process.platform === 'darwin') { // 如果是 mac 
    template.unshift({ // 添加
      label: 'DevDocs',
      submenu: [
        {
          role: 'about'
        },
        checkForUpdates,
        {
          type: 'separator'
        },
        ...preferences,
        {
          role: 'services',
          submenu: []
        },
        {
          type: 'separator'
        },
        {
          role: 'hide'
        },
        {
          role: 'hideothers'
        },
        {
          role: 'unhide'
        },
        {
          type: 'separator'
        },
        {
          role: 'quit'
        }
      ]
    })

```
    

### 编辑-朗读

``` js
// Edit menu.
    template[1].submenu.push( 
      {
        type: 'separator'
      },
      {
        label: 'Speech',
        submenu: [
          {
            role: 'startspeaking'
          },
          {
            role: 'stopspeaking'
          }
        ]
      }
    )

```

### 窗口-快捷键

``` js
    // Window menu.
    template[3].submenu = [
      {
        label: 'Close',
        accelerator: 'CmdOrCtrl+W',
        role: 'close'
      },
      {
        label: 'Minimize',
        accelerator: 'CmdOrCtrl+M',
        role: 'minimize'
      },
      {
        label: 'Zoom',
        role: 'zoom'
      },
      {
        type: 'separator'
      },
      {
        label: 'Bring All to Front',
        role: 'front'
      }
    ]
  } else {
      // 不是 mac
    template.push({
      label: 'Preferences',
      submenu: [...preferences[0].submenu, preferences[1], checkForUpdates]
    })
  }

```

### 帮助

``` js
  template.push({
    role: 'help',
    submenu: [
      {
        label: 'Report Issues',
        click() {
          shell.openExternal(
            'http://github.com/egoist/devdocs-desktop/issues/new'
          )
        }
      }
    ]
  })

  return Menu.buildFromTemplate(template) // 建立
}

module.exports = createMenu

```