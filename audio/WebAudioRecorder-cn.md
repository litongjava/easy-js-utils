# WebAudioRecorder 类使用指南

## 概述

`WebAudioRecorder` 类提供了一个用于在Web应用中录制音频的简单接口。它允许您启动、暂停、恢复和停止录音，获取录音数据，播放录音，以及将录音保存为WAV文件。
## 使用示例
使用WebAudioRecorder开发一个录音程序
```vue
<template>
<div>
  <audio ref="audio" controls></audio>
  <input type="button" value="录音" @click="startRecording"/>
  <input type="button" value="暂停" @click="pauseRecording"/>
  <input type="button" value="继续" @click="resumeRecording"/>
  <input type="button" value="停止" @click="stopRecording"/>
  <input type="button" value="连接" @click="linkRecording"/>
  <input type="button" value="保存" @click="saveRecording"/>
  <input type="button" value="上传" @click="uploadRecording"/>
  <div ref="text"></div>
</div>
</template>

<script>
import WebAudioRecorder from '../utils/WebAudioRecorder'
import axios from 'axios';

export default {
  name: "Recorder",
  data() {
    return {
      recorder: null
    };
  },
  methods: {
    startRecording() {
      WebAudioRecorder.get(rec => {
        this.recorder = rec;
        this.recorder.start();
      });
    },
    pauseRecording() {
      if (this.recorder) {
        this.recorder.pause();
      }
    },
    resumeRecording() {
      if (this.recorder) {
        this.recorder.resume();
      }
    },
    stopRecording() {
      if (this.recorder) {
        this.recorder.stop();
      }
    },
    linkRecording() {
      if (this.recorder) {
        this.recorder.link(this.$refs.audio);
      }
    },
    saveRecording() {
      if (this.recorder) {
        this.recorder.save();
      }
    },
    uploadRecording() {
      if (this.recorder) {
        const formData = new FormData();
        const blob = this.recorder.getBlob();
        formData.append('file', blob, 'recording.wav');

        axios.post("audio/save/", formData, {
          headers: {
            'Content-Type': 'multipart/form-data'
          }
        })
          .then(response => {
            console.log('上传成功', response.data);
          })
          .catch(error => {
            console.error('上传失败', error);
          });
      }
    }
  }
};
</script>

```
## 功能

- **开始录音**：启动音频捕获。
- **暂停录音**：暂时停止音频捕获，但不结束录音会话。
- **恢复录音**：在暂停后继续音频捕获。
- **停止录音**：结束录音会话，并准备音频数据。
- **播放录音**：播放已捕获的音频。
- **保存录音**：将录音以WAV格式保存到本地文件系统。

## 使用方法

### 初始化

首先，确保已将 `WebAudioRecorder.js` 导入到您的项目中。

```javascript
import WebAudioRecorder from './path/to/WebAudioRecorder.js';
```

然后，需要用户授权使用麦克风。一旦获得授权，可以创建一个 `WebAudioRecorder` 实例：

```javascript
navigator.mediaDevices.getUserMedia({ audio: true })
  .then(stream => {
    const recorder = new WebAudioRecorder(stream);
  })
  .catch(error => {
    console.error('授权失败：', error);
  });
```

### 录音

使用 `start` 方法开始录音：

```javascript
recorder.start();
```

如果需要暂停，可以调用 `pause` 方法：

```javascript
recorder.pause();
```

要恢复录音，使用 `resume` 方法：

```javascript
recorder.resume();
```

完成录音后，使用 `stop` 方法停止录音：

```javascript
recorder.stop();
```

### 播放和保存

要播放录制的音频，您需要一个 `<audio>` 元素。然后，使用 `play` 方法播放录音：

```html
<audio id="audioPlayback"></audio>
```

```javascript
const audioElement = document.getElementById('audioPlayback');
recorder.play(audioElement);
```

要下载录音，使用 `download` 方法，可以指定下载的文件名：

```javascript
recorder.download('myRecording.wav');
```

### 配合 `axios` 上传到服务器
#### 1. 获取录音数据
首先，使用 `WebAudioRecorder` 实例的 `getBlob()` 方法来获取录音的 Blob 对象。这通常在录音完全停止后进行。
#### 2. 使用 Axios 上传录音

一旦您有了录音的 Blob 对象，就可以使用 `axios` 将它上传到服务器。创建一个 `FormData` 对象并将 Blob 添加到其中，然后通过 `axios` 发送 POST 请求。

```javascript
import axios from 'axios';


const formData = new FormData();
const blob = this.recorder.getBlob();
formData.append('file', blob, 'recording.wav');

axios.post("/audio/save", formData, {
  headers: {
    'Content-Type': 'multipart/form-data'
  }
})
.then(response => {
  console.log('上传成功', response.data);
})
.catch(error => {
  console.error('上传失败', error);
});


```

在这个函数中，我们创建了一个 `FormData` 实例，并将录音 Blob 作为文件添加到其中。`axios.post` 方法用于向指定的 URL 发送包含录音文件的请求。

## 注意事项

- 确保在用户界面中处理好授权麦克风的流程。
- 录音操作依赖于浏览器的MediaStream API，不同浏览器的支持程度可能不同。
- 在使用录音功能之前，始终确保已经获得了用户的录音权限。
- `WebAudioRecorder` 类不包含对录音设备选择的支持，它使用默认的录音设备。

## 兼容性

由于这个类使用了一些现代Web API（如 `AudioContext` 和 `getUserMedia`），因此它可能不在所有浏览器中都受支持。建议在最新版本的主流浏览器（如Chrome、Firefox、Safari）中使用此类。
