# Desktop Recorder

デスクトップのウィンドウやスクリーンを録画するアプリです

## Demo

## 動作条件
macOS 10.12 Sierraでのみ動作確認をしています

## 使い方

#### 基本的な使い方
1. 左側のリストから録画したい画面を選択する
1. ツールバーの一番左のレコード開始ボタンを押す
1. 画面の録画が開始される
1. 録画が終了したら、ツールバーの一番左のレコード終了ボタンを押す
1. 録画の保存はレコード開始ボタンの右にある保存ボタンを押す

#### ショートカットキー
| ショートカットキー | 説明 |
| --------------- | -- |
| Command + P | 録画開始 |
| Command + E | 録画終了(アプリにフォーカスがなくても動作する) |
| Command + S | 保存 |
| Command + R | 画面更新 |

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:9080
npm run dev

# build electron application for production
npm run build


# lint all JS/Vue component files in `src/`
npm run lint

```

---

This project was generated with [electron-vue](https://github.com/SimulatedGREG/electron-vue)@[1c165f7](https://github.com/SimulatedGREG/electron-vue/tree/1c165f7c5e56edaf48be0fbb70838a1af26bb015) using [vue-cli](https://github.com/vuejs/vue-cli). Documentation about the original structure can be found [here](https://simulatedgreg.gitbooks.io/electron-vue/content/index.html).
