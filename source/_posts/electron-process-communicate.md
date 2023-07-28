---
layout: post
title: electron进程通信
date: 2022-10-11 18:00:53
tags: electron
---

# electron 进程间通信

## electron 为我们提供了 2 个 IPC(进程间通信)模块，称为`ipcMain`和`ipcRender`

ipcMain：主进程向渲染进程的异步通信

ipcRenderer：渲染进程向主进程发送消息

## 示例 1：渲染进程向主进程发送消息

渲染进程发消息：

```
import { ipcRenderer } from 'electron'
export default{
  methods: {
    minimizeWin() {
      ipcRenderer.send('window-min')
    }
  }
}
```

主进程（background.js）接收消息：

```
import { app, BrowserWindow, ipcMain } from 'electron'

// 主进程接收消息
ipcMain.on('window-min', () => {
  win.minimize()
})
```

## 示例 2：主进程向渲染进程发送消息

主进程（background.js）发送消息：

```
import { app, BrowserWindow, ipcMain } from 'electron'

ipcMain.on('window-destroy', () => {
  // 主进程发送消息
  win.webContents.send('window-destroy')
})
```

渲染进程接收消息

```
import { ipcRenderer } from 'electron'
export default{
  mounted() {
    ipcRenderer.on('window-destroy', (event, data) => {
      // do something
    })
  }
}
```

## 示例 3：双向通信

渲染进程：

```
import { ipcRenderer } from 'electron'
export default{
  data() {
    return {
      mountPath: ''
    }
  },
  methods: {
    openFilePicker() {
      // 向主进程发送消息
      ipcRenderer.send('openfile')
      // 接收主进程的消息
      ipcRenderer.once('openfile', (event, data) => {
        // do some thing
      })
    }
  }
}
```

主进程（background.js）：

```
import { app, BrowserWindow, ipcMain, dialog } from 'electron'
const os = require('os')

// 接收渲染进程的消息
ipcMain.on('openfile', (event) => {
  dialog.showOpenDialog({
    defaultPath: os.homedir(),
    properties: ['openDirectory']
  }).then(res => {
    event.sender.send('openfile', res.filePaths) // 向渲染进程发消息
  }).catch(err => {
    console.log(err)
  })
})
```

以上就是进程通信的所有内容，🤔 有问题欢迎指出...
