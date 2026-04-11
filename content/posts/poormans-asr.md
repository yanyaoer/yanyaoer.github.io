---
title: "poorman's ASR solution"
date: 2025-05-11T17:50:56+08:00
---


## Intro

大概20年h2期间，听闻feishu支持了会议自动转译字幕功能，but
当时私有化部署的版本一直没有支持，于是私下找了语音助手团队的同学企图蹭服务用于自动生成会议纪要，**懒即是原动力**

再 but 一下，出于服务成本考量没有蹭成功，后来对应的同学也陆续离职就没下文了

23年大模型开始各种火热起来，于是借助 openai 的 whisper
在本地陆续写了一些音频转文本的小任务，比如 yt-dlp 下载 bilibili
视频后转音频再转文本，个人习惯上阅读的效率还是高于视频形式的

- [scribe] https://gist.github.com/yanyaoer/5cc7b0dd6729f306ad3cb740d501cabd

另外结合语音识别和 LLM 的发散，也试着搓了点语音助手的尝试

- [voice assistant with computer-use] https://gist.github.com/yanyaoer/752a73d2d6df5a2aa0de179a0f68e8a1
- [hook for whisper llm-talk] https://github.com/yanyaoer/whisper.cpp/commit/da85a1b4791e4a7eeb5adb31d47d4c5e53618234

但是实时语音识别场景，用 whiper-stream / sense-voice
这类方案还是会有识别不准确、内容丢失的问题存在，调整 step/length
参数效果也不明显，vad 方案延迟也很高...


## New solution

上周忽然想起了 kaldi 项目也蛮经典了，于是就试着用 sherpa-[ncnn|onnx] 尝试了下 demo
sherpa-[ncnn|onnx]-microphone
流式识别输出的整体效果已经很接近把音频文件喂给大模型的质量

于是联系维护者 @csukuangfj
请教怎么提升识别准确率，大佬过于卷周末还给造了新方案和输出优化，甚至还录了 b
站视频 https://www.bilibili.com/video/BV1xcEMz8EsX/


{{< bilibili BV1xcEMz8EsX >}}


- [Add real-time speech recognition example for SenseVoice. (#2197)] https://github.com/k2-fsa/sherpa-onnx/commit/53518efd2fe70f49b86f180a4e5b49fdc374da82
- [Fix displaying streaming speech recognition results for Python. (#2196)] https://github.com/k2-fsa/sherpa-onnx/commit/4a833a754743076d68252731c681511e2d9390bd

另外也体验到 two-pass 双模型识别的效果，两种方式都能满足俺的使用场景🎉

- [Two-pass real-time speech recognition from a microphone with sherpa-onnx] https://github.com/k2-fsa/sherpa-onnx/blob/b269e5cccc03c3e61dd9f363bb7b64b5b0ce6c3a/python-api-examples/two-pass-speech-recognition-from-microphone.py

在大佬的基础示例上，简单补充点内容的保存

```diff
          result = result.lower().strip()
          display.update_text(result)
+          log(result)
          display.finalize_current_sentence()
          display.display()
        else:
          sample_buffers = []

        first_recognizer.reset(stream)


+ def log(result):
+   global file_handler
+   file_handler.write(f"{datetime.now().strftime('%Y-%m-%d %H:%M:%S')} - {result}\n")
+   file_handler.flush()


if __name__ == "__main__":
+  from datetime import datetime
+
+  ts = datetime.now().strftime("%y%m%d%H%M%S")
+  file_path = Path(f"~/Desktop/asr/2pass.{ts}.txt").expanduser()
+  file_handler = open(file_path, "a")
  try:
    main()
  except KeyboardInterrupt:
    print("\nCaught Ctrl + C. Exiting")
+  finally:
+    file_handler.close()

```


