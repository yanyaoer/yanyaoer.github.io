---
title: web audio
date: 2016-06-15 14:34:08
---

照文档撸了一下 AudioContext 可视化音频  
桌面浏览器上 Safari 9, Chrome stable 绘制正常  
移动端只有微信的 webview 能工作, 纯玩票叻
![wave](https://mdn.mozillademos.org/files/7977/wave.png)

ref:
<https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API/Visualizations_with_Web_Audio_API>

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, user-scalable=no">
    <title>AV</title>
  </head>
  <body>
    <canvas id="vis"></canvas>
    <audio id="av" src="YOUR_AUDIO_FILE"></audio>
    <script src="index.js"></script>
  </body>
</html>
```

```javascript
window.AudioContext = window.AudioContext || window.webkitAudioContext || window.mozAudioContext;

window.onload = function(){

  var canvas = document.getElementById('vis');
  var canvasCtx = canvas.getContext('2d');

  var isPlaying = true;
  var audio = document.getElementById('av');
  var audioCtx = new AudioContext();
  var analyser = audioCtx.createAnalyser();
  var audioSrc = audioCtx.createMediaElementSource(audio);
  audioSrc.connect(analyser);
  analyser.connect(audioCtx.destination);

  analyser.fftSize = 2048;
  var bufferLength = analyser.fftSize;
  dataArray = new Uint8Array(bufferLength);
  analyser.getByteTimeDomainData(dataArray);

  WIDTH = 500;
  HEIGHT = 150;
  function draw() {
    requestAnimationFrame(draw);
    if (!isPlaying) return;

    analyser.getByteTimeDomainData(dataArray);

    canvasCtx.fillStyle = 'rgb(200, 200, 200)';
    canvasCtx.fillRect(0, 0, WIDTH, HEIGHT);
    canvasCtx.lineWidth = 2;
    canvasCtx.strokeStyle = 'rgb(0, 0, 0)';
    canvasCtx.beginPath();

    var sliceWidth = WIDTH * 1.0 / bufferLength;
    var x = 0;

    for(var i = 0; i < bufferLength; i++) {

      var v = dataArray[i] / 128.0;
      var y = v * HEIGHT/2;

      if(i === 0) {
        canvasCtx.moveTo(x, y);
      } else {
        canvasCtx.lineTo(x, y);
      }

      x += sliceWidth;
    }

    canvasCtx.lineTo(canvas.width, canvas.height/2);
    canvasCtx.stroke();
  };

  draw();
  audio.play();

  document.body.onclick = function() {
    console.log(audio, isPlaying);
    if (isPlaying) {
      audio.pause();
      isPlaying = false;
    } else {
      audio.play();
      isPlaying = true;
    }
  }
}
```
