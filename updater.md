## updater


``` js
const isDev = require('electron-is-dev')
const log = require('electron-log') // 日志
const { autoUpdater } = require('electron-updater')

exports.init = () => {
  if (isDev || process.platform === 'linux') {
    // 如果是开发 或者 linux
    return
  }

  autoUpdater.logger = log
  autoUpdater.logger.transports.file.level = 'info'
  autoUpdater.checkForUpdates() // 检查 更新
}

```