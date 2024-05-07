---
layout: post
title: electron系统托盘
date: 2022-10-11 18:02:35
tags: electron
---

# electron 创建系统托盘

主进程（background.js）：

```
import { app, BrowserWindow, ipcMain, Tray, Menu, nativeImage } from 'electron'

let win
function createWindow() {
  // 创建窗口
  ...
  createTrayIcon()
}

// 创建系统托盘
let tray
function createTrayIcon() {
  const iconPath = nativeImage.createFromPath(path.join(__static, 'icons/cloud_client.png')) // __static指向public文件夹
  tray = new Tray(iconPath)
  const contextMenu = Menu.buildFromTemplate([
    {
      label: '产品介绍',
      click: () => {
        dialog.showMessageBoxSync({
          message: '云应用客户端' + config.version,
          detail: '*******',
          type: 'info'
        })
      }
    },
    {
      label: '打开 云应用',
      click: () => {
        win.show()
        win.setSkipTaskbar(false) // 显示任务栏图标
      }
    },
    {
      label: '退出 云应用',
      click: () => {
        win.setSkipTaskbar(true) // 隐藏任务栏图标
        win.destroy()
        app.quit()
      }
    }
  ])
  tray.setToolTip('云应用')
  tray.setContextMenu(contextMenu)
  tray.on('click', () => {
    win.show()
    win.setSkipTaskbar(false)
  })
}
```

![App Screenshot](./preview.png)

🤔 其他参数请移步[官网](https://www.electronjs.org/zh/docs/latest/api/tray)查看详细信息

💬 有问题欢迎指出...
