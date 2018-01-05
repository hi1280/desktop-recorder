<template>
  <div id="wrapper" class="window">
    <header class="toolbar toolbar-header">
      <div class="toolbar-actions">
        <div class="btn-group">
          <button class="btn btn-default" v-on:click="startRecord()" v-if="isStarted" v-bind:disabled="!stream">
            <span class="icon icon-record"></span>
          </button>
          <button class="btn btn-default" v-on:click="stopRecord()" v-if="!isStarted">
            <span class="icon icon-stop"></span>
          </button>
          <button class="btn btn-default" v-on:click="downloadVideo()" v-bind:disabled="!download">
            <span class="icon icon-download"></span>
          </button>
        </div>
        <button class="btn btn-default pull-right" v-on:click="refresh()">
          <span class="icon icon-arrows-ccw"></span>
        </button>
      </div>
    </header>
    <a id="download" v-bind:href="href" v-bind:download="download" style="display:none;">Download</a>
    <div class="window-content">
      <div class="pane-group">
        <div class="pane-sm sidebar">
          <ul class="list-group list-group-ex">
            <li class="list-group-item" v-bind:key="item.id" v-for="(item, idx) in sources" v-on:click="accessApproved(idx)">
              <div className='media-body'>
                <strong>{{ item.name }}</strong>
              </div>
              <img v-bind:src="item.thumbnail.toDataURL()"/>
            </li>
          </ul>
        </div>
        <div class="pane video">
          <video id="player"></video>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as electron from 'electron'
import * as moment from 'moment'
const SCREEN = 1
const {ipcRenderer} = electron
const {Menu} = electron.remote
export default {
  name: 'landing-page',
  components: {},
  methods: {
    accessApproved: function (idx) {
      const {width, height} = electron.screen.getPrimaryDisplay().workAreaSize
      navigator.mediaDevices.getUserMedia({
        audio: false,
        video: {
          mandatory: {
            chromeMediaSource: 'desktop',
            chromeMediaSourceId: this.sources[idx].id,
            minWidth: 640,
            maxWidth: width,
            minHeight: 480,
            maxHeight: height
          }
        }
      }).then(this.handleStream)
    },
    handleStream: function (stream) {
      const video = document.querySelector('#player')
      video.src = URL.createObjectURL(stream)
      video.autoplay = true
      video.controls = false
      this.stream = stream
      const menu = Menu.getApplicationMenu()
      menu.items[SCREEN].submenu.items[0].enabled = true
      menu.items[SCREEN].submenu.items[1].enabled = false
    },
    startRecord: function () {
      this.mediaRecorder = new MediaRecorder(this.stream)
      this.mediaRecorder.onstop = (event) => {
        this.href = URL.createObjectURL(new Blob(this.mediaRecorderChunks))
        this.download = `screen_${moment().format('YYYYMMDDHHmmss')}.webm`
        this.mediaRecorderChunks = []
        const video = document.querySelector('#player')
        video.src = this.href
        video.controls = true
        video.autoplay = false
        const menu = Menu.getApplicationMenu()
        menu.items[SCREEN].submenu.items[2].enabled = true
      }
      this.mediaRecorder.ondataavailable = (event) => {
        this.mediaRecorderChunks.push(event.data)
      }
      this.mediaRecorder.start()
      this.isStarted = false
      const menu = Menu.getApplicationMenu()
      menu.items[SCREEN].submenu.items[0].enabled = false
      menu.items[SCREEN].submenu.items[1].enabled = true
    },
    stopRecord: function () {
      if (this.mediaRecorder && this.mediaRecorder.state === 'recording') {
        this.mediaRecorder.stop()
        this.isStarted = true
        const menu = Menu.getApplicationMenu()
        menu.items[SCREEN].submenu.items[0].enabled = true
        menu.items[SCREEN].submenu.items[1].enabled = false
      }
    },
    downloadVideo: function () {
      let clicker = document.createEvent('MouseEvent')
      clicker.initEvent('click', false, true)
      const download = document.querySelector('#download')
      download.dispatchEvent(clicker)
    },
    refresh: function () {
      const desktopCapturer = electron.desktopCapturer
      desktopCapturer.getSources({ types: ['window', 'screen'] }, (error, sources) => {
        if (error) throw error
        this.sources = sources
      })
    }
  },
  data: function () {
    return {
      sources: [],
      stream: null,
      mediaRecorder: null,
      mediaRecorderChunks: [],
      href: '',
      download: '',
      isStarted: true
    }
  },
  created: function () {
    const {app} = electron.remote
    const template = [
      {
        label: '画面',
        submenu: [
          {
            label: '画面収録を開始',
            accelerator: 'CmdOrCtrl+P',
            click: function (menuItem, browserWindow, event) {
              if (browserWindow) {
                browserWindow.webContents.send('startRecord')
              } else {
                ipcRenderer.send('startRecord')
              }
            },
            enabled: false
          },
          {
            label: '画面収録を停止',
            accelerator: 'CmdOrCtrl+E',
            click: function (menuItem, browserWindow, event) {
              if (browserWindow) {
                browserWindow.webContents.send('stopRecord')
              } else {
                ipcRenderer.send('stopRecord')
              }
            },
            enabled: false
          },
          {
            label: '画面収録を保存',
            accelerator: 'CmdOrCtrl+S',
            click: function (menuItem, browserWindow, event) {
              if (browserWindow) {
                browserWindow && browserWindow.webContents.send('downloadVideo')
              } else {
                ipcRenderer.send('downloadVideo')
              }
            },
            enabled: false
          },
          {type: 'separator'},
          {
            label: '画面更新',
            accelerator: 'CmdOrCtrl+R',
            click: function (menuItem, browserWindow, event) {
              browserWindow && browserWindow.webContents.send('refresh')
            }
          }
        ]
      }
    ]
    if (process.platform === 'darwin') {
      template.unshift({
        label: app.getName(),
        submenu: [
          {
            label: `${app.getName()}について`,
            role: 'about'
          },
          {type: 'separator'},
          {
            label: 'サービス',
            role: 'services',
            submenu: []
          },
          {type: 'separator'},
          {
            label: `${app.getName()}を隠す`,
            role: 'hide'
          },
          {
            label: 'ほかを隠す',
            role: 'hideothers'
          },
          {
            label: 'すべてを表示',
            role: 'unhide'
          },
          {type: 'separator'},
          {
            label: `${app.getName()}を終了`,
            role: 'quit'
          }
        ]
      })
    }
    const menu = Menu.buildFromTemplate(template)
    Menu.setApplicationMenu(menu)
    electron.ipcRenderer.on('startRecord', (event, arg) => {
      this.startRecord()
    })
    electron.ipcRenderer.on('stopRecord', (event, arg) => {
      this.stopRecord()
    })
    electron.ipcRenderer.on('downloadVideo', (event, arg) => {
      this.downloadVideo()
    })
    electron.ipcRenderer.on('refresh', (event, arg) => {
      this.refresh()
    })
    this.refresh()
  }
}
</script>

<style>
@import "~photon/dist/css/photon.css";

.video {
  display: flex;
  justify-content: center;
  align-items: center;
  border-left: initial;
}

.list-group-ex {
  background-color: #f5f5f4;
  border-right: 1px solid #ddd;
}

</style>
