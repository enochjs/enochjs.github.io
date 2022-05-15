---
title: MIME
date: 2022-05-15 09:41:14
tags:
---

---

title: 媒体编码
date: 2022-05-14 19:39:40
tags:

---

#### MIME （Multipurpose Internet Mail Extensions）

1. 媒体类型（通常称为 Multipurpose Internet Mail Extensions 或 MIME 类型 ）是一种标准，用来表示文档、文件或字节流的性质和格式。
2. 通用结构

```bash
  type/subtype
```

3.  独立类型

```bash
 text                表明文件是普通文本                text/plain, text/html, text/css, text/javascript
 image               表明是某种图像。                 image/gif, image/png, image/jpeg, image/bmp, image/webp, image/x-icon, image/vnd.microsoft.icon
 audio               音频文件                        audio/midi, audio/mpeg, audio/webm, audio/ogg, audio/wav
 video               视频文件                        video/webm, video/ogg
 application         二进制数据                      application/octet-stream, application/pkcs12, application/vnd.mspowerpoint, application/xhtml+xml, application/xml,  application/pdf
```

4. Multipart 类型

```
multipart/form-data
multipart/byteranges
```

#### 视频文件的 mime

1. 如下例子

```
- audio/mp4; codecs="mp4v,mp4a.40.2"; profiles="isom,iso2,mp41"
- video/mp4; codecs="mp4a.40.2,avc1.64001f"; profiles="mp42,isom"
```

2. 语法介绍

- 第一部分为媒体类型
- codec，视频的压缩算法，可以有多个逗号分隔
- profile，其他描述信息，具体我也没看懂，可以有多个，逗号分隔

#### 市面常用的 codecs

1. H.264 Mpeg-4 AVC (高级视频编码)
2. H.265 HEVC (高效视频编码)
3. VP9 由谷歌开发作为一种开放和自由的视频压缩标准

#### 浏览器支持情况

- [H.264](https://caniuse.com/?search=H.264), 目前使用最广
- [H.265](https://caniuse.com/?search=H.265)
- [vp9](https://caniuse.com/?search=VP9)

#### js 如何查看视频编码

- [mp4box](https://github.com/gpac/mp4box.js)
- 代码示例

```typescript
// 浏览器
const arrayBuffer = await file.arrayBuffer();
// nodejs
// const buffer = await fs.promises.readFile(video_path);
// const arrayBuffer = new Uint8Array(buffer).buffer
arrayBuffer.fileStart = 0;
return new Promise((resolve, reject) => {
  try {
    var mp4BoxFile = MP4Box.createFile();
    mp4BoxFile.onError = () => reject(false);
    mp4BoxFile.onReady = (info: any) => {
      console.log("info", info.mime);
      const result = this.getCodecValid(info.mime);
      resolve(result);
    };
    mp4BoxFile.appendBuffer(arrayBuffer);
    mp4BoxFile.flush();
  } catch (error) {
    reject(error);
  }
});
```

#### 视频转码

- [fluent-ffmpeg](https://github.com/fluent-ffmpeg/node-fluent-ffmpeg)
- [@ffmpeg-installer/ffmpeg](https://github.com/kribblo/node-ffmpeg-installer)
- 代码示例

```typescript
import ffmpeg from "fluent-ffmpeg";
import { path as ffmpegPath } from "@ffmpeg-installer/ffmpeg";
ffmpeg.setFfmpegPath(ffmpegPath);

ffmpeg(inputPath)
  .videoCodec("libx264") // Codec  可调用命令 ffmpeg -codecs 查看所有支持的格式
  .inputFPS(25) // FPS
  .videoBitrate(2000) // BPS
  .on("error", function (err) {
    console.log("err", err);
  })
  .on("end", async function () {
    console.log("Processing finished !");
  })
  .save(outputPath);
```

### 相关链接

[MimeCodec](https://www.jackpu.com/web-video-mimetype-jiu-jing-dai-biao-shi-yao-yi-si/)
[JS 获取 Codec](https://www.jackpu.com/shi-yong-js-huo-qu-shi-pin-codec/)
[codec](https://datatracker.ietf.org/doc/html/rfc6381#section-3.3)
[知乎文章](https://zhuanlan.zhihu.com/p/71928833)
