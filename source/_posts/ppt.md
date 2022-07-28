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
import ffmpegPath from "@ffmpeg-installer/ffmpeg";
import decompress from "decompress";
import archiver from "archiver";
import dayjs from "dayjs";

const _ffmpegPath = ffmpegPath.path.replace("app.asar", "app.asar.unpacked");

ffmpeg.setFfmpegPath(_ffmpegPath);

const VIDEO_TYPE = new Array(
  ".avi",
  ".wmv",
  ".mpg",
  ".mpeg",
  ".mov",
  ".rm",
  ".ram",
  ".swf",
  ".flv",
  ".mp4",
  ".wma",
  ".rm",
  ".rmvb",
  ".flv",
  ".mpg",
  ".mkv"
);

export class ParseFile {
  zipDirectory(sourceDir: string, outPath: string) {
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

  public translateVideo(filePath: string, outDir?: string): Promise<string> {
    const inputPath = filePath;
    const pathDir = path.dirname(filePath);
    const fileName = path.basename(filePath, path.extname(filePath));

    const outputPath = path.resolve(
      outDir || pathDir,
      `./${fileName}_${dayjs().format("YYYY_MM_DD_HH_mm_ss")}`
    );
    const dest = outputPath + ".mp4";
    return new Promise((resolve, reject) => {
      ffmpeg(inputPath)
        .videoCodec("libx264")
        // 解决音视频不同步问题
        .audioCodec("copy)
        .inputFPS(25)
        .videoBitrate(2000)
        .on("error", function (err) {
          console.log("err", err);
          reject(err);
        })
        .on("end", async function () {
          if (!outDir) {
            await fs.promises.unlink(inputPath);
            await fs.promises.rename(dest, `${pathDir}/${fileName}.mp4`);
            resolve(`${pathDir}/${fileName}.mp4`);
          } else {
            resolve(dest);
          }
          console.log("Processing finished !");
        })
        .save(dest);
    });
  }

  public async replaceVideo(
    basePath: string,
    files: decompress.File[],
    sourceName: string,
    targetName: string
  ) {
    let total = files.length;
    while (total) {
      const file = files[total - 1];
      if (
        file &&
        /slide.*.xml.rels/.test(file.path) &&
        file.data.toString().indexOf(sourceName) !== -1
      ) {
        const reg = new RegExp(sourceName, "g");
        const data = await fs.promises.readFile(path.join(basePath, file.path));

        await fs.promises.writeFile(
          path.join(basePath, file.path),
          data.toString().replace(reg, targetName)
        );
      }
      total -= 1;
    }
  }

  public async translatePPT(
    filePath: string,
    outDir?: string
  ): Promise<string> {
    const fileBuffer = await fs.promises.readFile(filePath);
    const time = dayjs().format("YYYY_MM_DD_HH_mm_ss");
    let temp = path.join(__dirname, "temp", `${time}`);
    if (outDir) {
      temp = path.join(outDir, "temp", `${time}`);
      await fs.promises.mkdir(temp, { recursive: true });
    }

    const files = await decompress(fileBuffer, temp);
    let count = files.length;
    let suffix = path.extname(filePath);
    while (count) {
      const file = files[count - 1];
      if (file) {
        const fileName = path.basename(file.path);
        const ext = path.extname(fileName);

        if (VIDEO_TYPE.includes(ext)) {
          console.log(
            "VIDEO_TYPE",
            file.path,
            `${path.dirname(file.path)}.mp4`
          );
          await this.translateVideo(path.join(temp, file.path));
          if (ext !== ".mp4") {
            await this.replaceVideo(
              temp,
              files,
              fileName,
              `${path.basename(file.path, ext)}.mp4`
            );
            // 遇到视频需要改后缀的，设置为ppt后缀，提示用户手动转为pptx
            suffix = ".ppt";
          }
        }
      }
      count -= 1;
    }

    const fileName = path.basename(
      filePath,
      filePath.slice(filePath.lastIndexOf("."))
    );
    const outputPath = path.resolve(
      outDir || __dirname,
      `./${fileName}_${time}${suffix}`
    );
    await this.zipDirectory(path.resolve(temp), outputPath);
    try {
      await fs.promises.rm(temp, { recursive: true });
      return outputPath;
    } catch (error) {
      console.log("translate ppt error", error);
      return outputPath;
    }
  }
}

const parseFile = new ParseFile();

export default parseFile;
```

### 参考链接

[mime](../mime)
