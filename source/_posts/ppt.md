---
title: ppt
date: 2022-05-15 10:17:03
tags:
---

### 如何获取 ppt 内的所有文件

- 可以看下[microsoft](https://support.microsoft.com/en-us/office/extract-files-or-objects-from-a-powerpoint-file-85511e6f-9e76-41ad-8424-eab8a5bbc517)中的介绍，一下为其中一段话

```
If you want to separately use files or objects from a PowerPoint presentation,
such as videos, photos, or sounds,
you can extract them by converting the presentation to a “zipped” file folder.
Note, however, that you can't extract PDFs or .dotx files.

就是说，如果你想看ppt内的所有文件，你可以将ppt的后缀名改为 .zip, 然后解压就可以看到所有内容了

```

- 详情截图
  ![](ppt.jpg)

### 如何修改 ppt

- 解压 zip 包
- 修改文件
- 压缩成 zip
- 修改文件后缀

### 修改 ppt 中的视频 demo

```typescript
import path from "path";
import fs from "fs";
import ffmpeg from "fluent-ffmpeg";
import { path as ffmpegPath } from "@ffmpeg-installer/ffmpeg";
import decompress from "decompress";
import archiver from "archiver";

ffmpeg.setFfmpegPath(ffmpegPath);

const VIDEO_TYPE = new Array(
  "avi",
  "wmv",
  "mpg",
  "mpeg",
  "mov",
  "rm",
  "ram",
  "swf",
  "flv",
  "mp4",
  "wma",
  "rm",
  "rmvb",
  "flv",
  "mpg",
  "mkv"
);

export class ParseFile {
  // 压缩
  private zipDirectory(sourceDir: string, outPath: string) {
    const archive = archiver("zip", { zlib: { level: 9 } });
    const stream = fs.createWriteStream(outPath);
    return new Promise((resolve, reject) => {
      archive
        .directory(sourceDir, false)
        .on("error", (err) => reject(err))
        .pipe(stream);
      stream.on("close", () => resolve(true));
      archive.finalize();
    });
  }

  // 将video 转为H.264
  public translateVideo(filePath: string) {
    const inputPath = filePath;
    const fileNameIndex = filePath.lastIndexOf("/");
    const outputPath =
      inputPath.slice(0, fileNameIndex) +
      "/output_" +
      inputPath.slice(fileNameIndex + 1);
    return new Promise((resolve, reject) => {
      ffmpeg(inputPath)
        .videoCodec("libx264")
        .inputFPS(25)
        .videoBitrate(2000)
        .on("error", function (err) {
          console.log("err", err);
          reject(err);
        })
        .on("end", async function () {
          await fs.promises.unlink(inputPath);
          await fs.promises.rename(outputPath, inputPath);
          console.log("Processing finished !");
          resolve("Processing finished !");
        })
        .save(outputPath);
    });
  }

  // 转ppt
  public async translatePPT(filePath: string) {
    const fileBuffer = await fs.promises.readFile(filePath);
    const time = new Date().getTime();
    const temp = path.resolve(__dirname, `./temp/${time}`);

    const files = await decompress(fileBuffer, temp);

    let count = files.length;
    while (count) {
      const file = files[count - 1];
      const fileName = file.path.slice(file.path.lastIndexOf("/") + 1);
      const ext = fileName.slice(fileName.lastIndexOf(".") + 1);
      if (VIDEO_TYPE.includes(ext)) {
        await this.translateVideo(`${temp}/${file.path}`);
      }
      count -= 1;
    }
    console.log(path.resolve(__dirname, `./output/${time}.ppt`));
    await this.zipDirectory(
      path.resolve(temp),
      path.resolve(__dirname, `./output_${time}.ppt`)
    );
    try {
      await fs.promises.rm(temp, { recursive: true });
    } catch (error) {}
  }
}

const parseFile = new ParseFile();

// parseFile.translatePPT(
//   path.resolve(__dirname, "your_ppt_path.ppt")
// );

export default parseFile;
```

### 参考链接

[mime](../mime)
