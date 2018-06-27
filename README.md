# imageresizer

## 工具地址：
- 项目浏览器访问项目：

- 用户本地生成生成。
  
``` bash
# install dependencies （安装依赖）
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

```

## 需求来源：
> 入职以来，公司HR小姐姐经常给我一些新员工入职的一寸照片，让我使用PhotoShop修成特定尺寸（像素）/ 特定大小（KB）的文件。作为一个码农，实在收不了重复的工作，于是做了这个工具来满足HR小姐姐的使用需求。



## 思路：

1. 通过 `<input type="file">` 实现本地文件上传。
2. 通过 浏览器`FileReader ` 实现将图片转为 `Base64`编码。
3. 将 `Base64`图片绘制到 `canvas`画布
4. 通过 `input` 获取用户输入，设置canvas画布尺寸（px）,这将是图片生成时的宽高（px）。
5. 通过调整按钮调整图像以充满画布。没有被填充的区域生成图片的时候将为黑色。
6. 通过 `canvas` 的 `toDataUrl`方法 将canvas画布转为`base64`图像。
7.  `new Image()`生成image对象设定`src`为canvas画布生成的`base64`图像，并渲染至`DOM`。
8. 用户下载，生成符合预期宽高和文件大小的文件。



 


