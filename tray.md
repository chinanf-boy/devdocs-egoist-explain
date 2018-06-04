## tray

添加, 系统菜单的图标和应用图片, 主要是 `windows`/`linux`

<!-- START doctoc -->
<!-- END doctoc -->

``` js
const path = require('path')
const { app, Menu, Tray } = require('electron')

let tray = null

exports.create = win => {
  if (process.platform === 'darwin' || tray) {
      // mac 返回
    return
  }

  const iconPath = path.join(__dirname, 'static/tray.png')

  const toggleWin = () => { // 主要是 主窗口的 显示/隐藏
    if (win.isVisible()) {
      win.hide()
    } else {
      win.show()
    }
  }

  const contextMenu = Menu.buildFromTemplate([
    {
      label: 'Toggle',
      click() {
        toggleWin()
      }
    },
    {
      type: 'separator'
    },
    {
      role: 'quit'
    }
  ])

  tray = new Tray(iconPath) // 图标
  tray.setToolTip(`${app.getName()}`) // 名称
  tray.setContextMenu(contextMenu) // 菜单内容
  tray.on('click', toggleWin) // 点击图标就是 显示和隐藏的关系
}

```