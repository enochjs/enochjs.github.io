---
title: 解决canvas绘图模糊问题
date: 2020-09-03 10:26:15
tags:
---

### 相关概念
1. 像素
  - **像素，图像显示的基本单位**，英文pixel(picture + element)
  - 显示器的一个像素就是一个方格，电脑显示器、手机显示器，都是由一个个方格像素点组成，类似于显示的基本单位
  - 一个像素点，同一时刻只能显示一种颜色，这个颜色由三原色调配组成
  - 人们常说的相机有800w像素，就是指相机拍的一张照片上有800w个像素点。同样大小的照片，像素越高，就越清晰、精细
2. 分辨率（解析度）
  - **图像分辨率(ppi)：每英寸的像素数**
    如：一幅图宽8英寸、高6英寸，分辨率为100PPI，如果保持图像文件的大小不变，也就是总的像素数不变，将分辨率降为50PPI，在宽高比不变的情况下，图像的宽将变为16英寸、高将变为12英寸。
  - pt， point，点，印刷行业常用单位，等于1/72英寸。
  - **输出分辨率(dpi)：设备输出图像时每英寸可产生的点数**（打印时使用）, 电子设备中 dpi===ppi
  - **dp，dip， Density-independent pixel，安卓开发用的长度单位。**以160ppi为标准，和iPhone的scale差不多的意思。安卓用dp适配，系统会自动将dp转换为px。当屏幕像素点密度为160ppi时，  1dp=1px。

### canvas 高dpi下模糊的原因
  1. canvas宽高和样式宽高,如下代码：
    ```html
      <canvas width="320" height="150" style="width: 320px; height: 150px"></canvas>
    ```

    - width、height代表canvas实际大小，这个大小决定了canvas像素大小
    - style width、height 代表css像素（对应设备）
    问题解析:
    如上示例canvas像素为 320 * 150, 在dpi为1的设备上一比一展示是正常的，但是在**在retina屏幕下，1个canvas像素（或者说是1个位图像素）将会填充4个物理像素，由于单个位图像素不可以再进一步分割，所以只能就近取色，从而导致图片模糊。**
  
  2. 解决办法
    - 模糊是像素不够就近取色导致的，所以解决方案就是补充像素，将canvas width、height 扩大dpr倍，里面图形也放大dpr倍，这样在retina屏幕下图形也能一个像素点对应一个像素点
    ```html
      <!DOCTYPE html>
      <html lang="en">
      <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
        <style>
          div {
            margin-left: 0;
          }
          canvas {background: #eee;}
        </style>
      </head>
      <body>
        <div>
          <canvas id="myCanvas"></canvas>
        </div>
        <script>
          function setupCanvas(canvas, width, height) {
            // 获取devicePixelRatio， 物理像素/css像素 比
            const dpr = window.devicePixelRatio || 1
            const rate = 1/dpr
            const ctx = canvas.getContext('2d');
            // canvas宽高设置 dpr 倍
            canvas.width = width * dpr;
            canvas.height = height * dpr;
            // canvas放大，内容也放大dpr倍
            ctx.scale(dpr, dpr);
            // 设置指定canvas css宽高
            canvas.style.width = `${width}px`
            canvas.style.height = `${height}px`
            // 或者通过transform 改变canvas css大小
            // 设置canvas transformOrigin
            // canvas.style.transformOrigin = '0px 0px'
            // 由于canvas放大了dpr倍，需要在显示的时候，css缩放为的1/dpr
            // canvas.style.transform = `scale(${rate}, ${rate})`

            return ctx;
          }
          const canvas = document.querySelector('#myCanvas')
          const rect = canvas.getBoundingClientRect();
          const ctx = setupCanvas(canvas, rect.width, rect.height);
          ctx.strokeRect(30,30,100, 100);
          ctx.font = "30px Arial";
          ctx.fillText("Demo!", 35, 85);
        </script>
        
      </body>
      </html>
    ```

    ### 参考链接
    - [Canvas在移动端绘制模糊的原因与解决办法](http://www.fly63.com/article/detial/3091)
    - [Fixing HTML5 2d Canvas Blur](https://medium.com/wdstack/fixing-html5-2d-canvas-blur-8ebe27db07da)
    - [High DPI Canvas](https://www.html5rocks.com/en/tutorials/canvas/hidpi/)