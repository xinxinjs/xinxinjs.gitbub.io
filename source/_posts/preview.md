---
title: 浏览器在线预览文件（PDF,txt,html,ofd,docx,pptx,jpg,jpeg,png...等）
date: 2020-11-09 10:19:59
tags:
categories:
---

最近做项目，接到一个需求：要求预览上传的文件，大概包括以下格式：PDF,txt,html,ofd,docx,pptx,jpg,jpeg,png...

首先参考前人（同事）的经验，将要预览的文件进行了分类：

- Office类型的

- 图片类型的

- 其他类型：PDF，ofd，html，txt

项目使用的是react + hooks + ant design搭建，我搭建了一个demo来演示预览功能

## Office类型的

根据分类，最简单的是Office类型的文件预览，微软的提供了一个官方的预览方法，参数url是你的附件地址，直接在浏览器里打开这个地址，再后面拼上文件的地址就可以了，内嵌在iframe也可以

```javascript
`https://view.officeapps.live.com/op/view.aspx?src=${url}`
```

## 图片类型的

图片类型的是最简单的，直接写进img标签里就可以

```javascript
<img src={url} style={{width: '1000px', margin: '0 auto'}} />
```

## PDF类型

比较麻烦的是剩下几个类型的，第一个先说PDF格式的预览

经过调查和研究最终选择了现有的一个库pdfjs-dist来实现该功能

实现步骤：

安装：`npm install pdfjs-dist --save`

在项目中引用

```tsx
import * as PDFJS from 'pdfjs-dist';
import pdfjsWorker from 'pdfjs-dist/build/pdf.worker.entry';

PDFJS.GlobalWorkerOptions.workerSrc = pdfjsWorker;

const pdfurl = '';

function PreView() {
  const containerRef = useRef();

  // 设置canvas，将PDF文件分页用canvas绘制出来，并渲染在页面
  const openPage = (pdf: any, pageNumber: number) => {
    pdf.getPage(pageNumber).then((pdfPage: any) => {
      var viewport = pdfPage.getViewport({ scale });
      const canvas = document.createElement('canvas');
      canvas.id = 'test';
      canvas.width = viewport.width;
      canvas.height = viewport.height;
      var ctx = canvas.getContext("2d");
      var renderTask = pdfPage.render({
        canvasContext: ctx,
        viewport: viewport,
      });
      containerRef.current.appendChild(canvas);
      return renderTask.promise;
    });
  }

  // PDFJS.getDocument（url）获取PDF，解析PDF
  const readPDF = async () => {
    const loadingTask = await PDFJS.getDocument(pdfurl);
    loadingTask.promise.then((pdf: any) => {
      const pageNum = pdf.numPages;
      for (let i = 1; i <= pageNum; i++) {
        openPage(pdf, i);
      }
    });
  }

  useEffect(() => {
    readPDF();
  }, []);

  return (
    <>
      <div ref={containerRef} />
    </>
  )
}

export default PreView;

```

## ofd格式

ofd格式的文档，我还是第一次听说，我们这次使用ofd格式的文档主要是用于发票的上传和预览

先百科一下，顺便自己长点知识：OFD格式是我国自主可控的电子文件版式文档格式。OFD版式文件，版面固定、不跑版、所见即所得，可以视为计算机时代的“数字纸张”；是电子文档发布、数字化信息传播和存档的理想文档格式。（来自知乎复制黏贴）

OFD格式具有以下优势：

1. 产权属于自主产权。

2. 具有便携性：文件小，可压缩比率大。测试显示生成的文件体量比PDF还要小。

3. 具有开放性：易于入门，对于使用者来说更具开放性。

4. 具有扩展性：预留了可扩展入口和自定义标引，设置了非接触式引用机制，为特性化提供支持。

5. 呈现效果与设备无关，在各种设备上阅读、打印或印刷时，版面固定、不跑版。

6. 应用广泛：无论是电子商务、电子公务，还是信息发布、文件交换，档案管理等都需要版式文档的技术支持。

### 如何预览

经过查找，找到了ofd.js工具库来实现ofd文件的预览，现在项目里安装一下

`npm i ofd.js`

然后在页面中引入并使用

```tsx
function PreView() {
  const { parseOfdDocument, renderOfd } = require('ofd.js');
  const contentDiv = useRef();
  const ofdRef = useRef();

  useEffect(() => {
    return () => {
      delete Array.prototype.pipeline;
    }
  }, []);

  const fileChanged = (e) => {
    const file = ofdRef.current.input.files[0];
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = function (e) {
      const ofdBase64 = e.target.result.split(',')[1];
    }
    // 核心方法，file支持本地文件、二进制或者url
    parseOfdDocument({
      ofd: file,
      success(res) {
        console.log(res);
        //输出ofd每页的div
        const divs = renderOfd(900, res[0]);
        for (const div of divs) {
          contentDiv.current.appendChild(div);
        }
      },
    });
  }


  return (
    <>
      <Input type="file" accept=".ofd" onChange={(e) => fileChanged(e)} ref={ofdRef} />
      <div ref={contentDiv}></div>
    </>
  )
}

export default PreView;
```

在这里为什么要使用require('ofd.js')的方式引入呢？原因是，ofd.js的源码里有这样子一段代码：

```javascript
'use strict';
Array.prototype.pipeline = async function(callback){
  if (null === this || 'undefined' === typeof this) {
    // At the moment all modern browsers, that support strict mode, have
    // native implementation of Array.prototype.reduce. For instance, IE8
    // does not support strict mode, so this check is actually useless.
    throw new TypeError(
      'Array.prototype.pipeline called on null or undefined');
  }
  if ('function' !== typeof callback) {
    throw new TypeError(callback + ' is not a function');
  }
  var index, value,
    length = this.length >>> 0;
  for (index = 0; length > index; ++index) {
    value = await callback(value, this[index], index, this);
  }
  return value;
};

let _pipeline = function(...funcs){
  return funcs.pipeline((a, b) => b.call(this, a));
}

export const pipeline = _pipeline;
```

这段代码在array的原型链上添加了一个方法，如果直接放在顶层引用，可能会报错

最后在组件卸载的时候，最好手动删除一下这个添加在原型链上的方法，我已经在上面的代码里删除了

```tsx
useEffect(() => {
    return () => {
      delete Array.prototype.pipeline;
    }
  }, []);
```

## txt，html

这两种文件预览的处理方式差不多，先看一下代码

```tsx
const url = 'xx.txt'; // 或者xx.html
function Index() {
  const [txt, setTxt] = useState([]);

  const htmlRef = useRef();
  let fileType = url ? url.split('.').pop() : '';

  const viewTxt = () => {
    const xhr = new XMLHttpRequest();
    xhr.open('get', url);
    xhr.responseType = 'blob';
    xhr.onload = function() {
      var Blob = xhr.response;
      const reader = new FileReader();
      reader.readAsText(Blob, 'utf-8');
      reader.onload = (result: any) => {
        let targetNum = result.target.result;
        const changeString = targetNum.replace(/\r/g, '').split('\n');
        setTxt(changeString); // 存进state，用于txt文档的展示
        if(fileType == 'html') {
          htmlRef.current.innerHTML = targetNum; //使用innerHTML把html字符串加入到dom中去
        }
      };
    };
    xhr.send();
  }

  useEffect(() => {
    if(fileType == 'txt' || fileType == 'html') viewTxt();
  }, [url]);

  return (
    <div style={{ background: '#fff' }}>
      txt文件预览： 
      <div style={{width: '100%', height: '100%'}}>{txt}</div>
      html文件预览
      <div ref={htmlRef} style={{width: '100%', height: '100%'}} />
    </div>
  );
};

export default Index;

```

大概的代码就是这样子，我拿到文件的url地址，通过XMLHttpRequest去请求这个资源，然后设置responseType = 'blob';

拿到返回值之后再通过FileReader的方式读取内容，然后用自己的方式展示在页面上

txt文件直接拿到内容就可以展示，最多处理一下换行什么的

html格式的比较特殊，因为需要保留人家html的样式和布局，所以使用innerHTML的方式把字符串插入到dom中