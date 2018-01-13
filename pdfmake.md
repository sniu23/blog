# PDFMAKE

> 客户端/服务器端 PDF 打印的纯 JavaScript 包。

## 安装

``` bash
# 直接安装：
npm install pdfmake

# 或者源码安装： 
git clone https://github.com/bpampuch/pdfmake.git
cd pdfmake
npm install
npm run build
```

## 服务器端使用

```js
var fonts = {
	Roboto: {
		normal: 'fonts/Roboto-Regular.ttf',
		bold: 'fonts/Roboto-Medium.ttf',
		italics: 'fonts/Roboto-Italic.ttf',
		bolditalics: 'fonts/Roboto-MediumItalic.ttf'
	}
};

var PdfPrinter = require('../src/printer');
var printer = new PdfPrinter(fonts);
var fs = require('fs');

var docDefinition = {
	content: [
		'First paragraph',
		'Another paragraph, this time a little bit longer to make sure, this line will be divided into at least two lines'
	]
};

var pdfDoc = printer.createPdfKitDocument(docDefinition);
pdfDoc.pipe(fs.createWriteStream('pdfs/basics.pdf'));
pdfDoc.end();
```

## 客户端使用

```html
<!doctype html>
<html lang='en'>
<head>
  <meta charset='utf-8'>
  <title>my first pdfmake example</title>
  <script src='build/pdfmake.min.js'></script>
  <script src='build/vfs_fonts.js'></script>
</head>
<body>
```

```js
require('pdfmake/build/pdfmake.js'); 
require('pdfmake/build/vfs_fonts.js');

var docDefinition = { content: 'This is an sample PDF printed with pdfMake' };
// download the PDF 
// [defaultFileName, cb, options]
pdfMake.createPdf(docDefinition).download('optionalName.pdf');
// open the PDF in a new window
// [options, window]
pdfMake.createPdf(docDefinition).open();
// print the PDF
pdfMake.createPdf(docDefinition).print();

//Asynchronous
$scope.generatePdf = function() {
  // create the window before the callback
  var win = window.open('', '_blank');
  $http.post('/someUrl', data).then(function(response) {
    // pass the "win" argument
    pdfMake.createPdf(docDefinition).open({}, win);
  });
};

$scope.generatePdf = function() {
  // create the window before the callback
  var win = window.open('', '_blank');
  $http.post('/someUrl', data).then(function(response) {
    // pass the "win" argument
    pdfMake.createPdf(docDefinition).print({}, win);
  });
};

// open/print in same window
pdfMake.createPdf(docDefinition).open({}, window);
pdfMake.createPdf(docDefinition).print({}, window);

// Put the PDF into your own page as URL data
const pdfDocGenerator = pdfMake.createPdf(docDefinition);
pdfDocGenerator.getDataUrl((dataUrl) => {
  const targetElement = document.querySelector('#iframeContainer');
  const iframe = document.createElement('iframe');
  iframe.src = dataUrl;
  targetElement.appendChild(iframe);
});

// Get the PDF as base64 data
const pdfDocGenerator = pdfMake.createPdf(docDefinition);
pdfDocGenerator.getBase64((data) => {
	alert(data);
});

// Get the PDF as buffer
const pdfDocGenerator = pdfMake.createPdf(docDefinition);
pdfDocGenerator.getBuffer((buffer) => {
	// ...
});

// Get the PDF as Blob
const pdfDocGenerator = pdfMake.createPdf(docDefinition);
pdfDocGenerator.getBlob((blob) => {
	// ...
});

// Using javascript frameworks
var pdfMake = require('pdfmake/build/pdfmake.js');
var pdfFonts = require('pdfmake/build/vfs_fonts.js');
pdfMake.vfs = pdfFonts.pdfMake.vfs;
// or
import pdfMake from "pdfmake/build/pdfmake";
import pdfFonts from "pdfmake/build/vfs_fonts";
pdfMake.vfs = pdfFonts.pdfMake.vfs;

// plus adding the config in webpack
// if throwed error Cannot read property 'TYPED_ARRAY_SUPPORT' of undefined
exclude: [ /node_modules/, /pdfmake.js$/ ]
 ```
 [后台使用 express 的例子][express]

## 文档定义对象

`元素`：

- content
    - text
    - image
    - columns
    - table
        - body
    - ul/ol
    - stack
    - pageReference
    - textReference
    - qr
    - canvas
    - toc
- styles
- defaultStyle
- images
- background [Function]
- watermark

`属性`：

- id: 'header1'
- width: 100 / '50%' / 'auto' / '*'
- height: 100
- absolutePosition: {x: 100, y: 100}
- fit: [100, 100]
- alignment: 'justify' / 'right'
- style: 'header' / ['quote', 'small']
- reversed: true
- start: 50
- counter: 10
- type / listType: 'none' / 'square' / 'circle' / 'lower-alpha' / 'upper-alpha' / 'lower-roman' / 'upper-roman'
- type: 'rect' / 'polyline' / 'line' / 'ellipse'
- separator: ')' / ['(', ')']
- markerColor: 'red'
- color: 'black'
- opacity: 0.3
- pageBreak: 'after' / 'before'
- fontSize: 18,
- bold: true,
- italics: true
- margin: [0, 0, 0, 10] / [0, 20] / 100
- columnGap: 20
- layout： 
- colSpan：
- border:
- foreground: 'red', 
- background: 'yellow'
- decoration: 'underline' / 'lineThrough' / 'overline'
- decorationStyle: 
- decorationColor:
- preserveLeadingSpaces: true
- fontFeatures
- ……


`注意`:

- column 样式会被继承和覆盖
- margin 不会被继承， 只能显示定义

## 段落

```js
var docDefinition = {
  content: [
    // 1. 不需要样式时：通过 字符串 来定义段落
    'This is a standard paragraph, using default style',
    // 2. 通过使用 { text: '...' } 这样的 对象 来定义段落的整体样式
    { text: 'This paragraph will have a bigger font', fontSize: 15 },
    // 3. 使用数组来代替字符串，可以定义段落中的个别部分样式
    {
      text: [
        'This paragraph is defined as an array of elements to make it possible to ',
        { text: 'restyle part of it and make it bigger ', fontSize: 15 },
        'than the rest.'
      ]
    }
  ]
};
```

## 样式字典

```js
var docDefinition = {
  content: [
    { text: 'This is a header', style: 'header' },
    'No styling here, this is a standard paragraph',
    { text: 'Another text', style: 'anotherStyle' },
    { text: 'Multiple styles applied', style: [ 'header', 'anotherStyle' ] }
  ],

  styles: {
    header: {
      fontSize: 22,
      bold: true
    },
    anotherStyle: {
      italics: true,
      alignment: 'right'
    }
  }
};
// 通过例子，你可以深度了解样式的嵌套和覆盖
```

## 列

```js
var docDefinition = {
  content: [
    'This paragraph fills full width, as there are no columns. Next paragraph however consists of three columns',
    {
      columns: [
        {
          // auto-sized columns 基于内容调整列宽
          width: 'auto',
          text: 'First column'
        },
        {
          // star-sized columns 如果有多个，列宽会等分剩余空间
          width: '*',
          text: 'Second column'
        },
        {
          // fixed width 
          width: 100,
          text: 'Third column'
        },
        {
          // % width
          width: '20%',
          text: 'Fourth column'
        }
      ],
      // 可选： 列与列间隙
      columnGap: 10
    },
    'This paragraph goes below all columns and has full width'
  ]
};
// 列对象不仅仅可以是 字符串
```

## 表

```js
var docDefinition = {
  content: [
    {
      layout: 'lightHorizontalLines', // optional
      table: {
        // headers 如果表跨页，表头将重复显示
        // 你可以申明有多少行被视作表头
        headerRows: 1,
        widths: [ '*', 'auto', 100, '*' ],
        body: [
          [ 'First', 'Second', 'Third', 'The last one' ],
          [ 'Value 1', 'Value 2', 'Value 3', 'Value 4' ],
          [ { text: 'Bold value', bold: true }, 'Val 2', 'Val 3', 'Val 4' ]
        ]
      }
    }
  ]
};

//表布局：必须在 pdfMake.createPdf(docDefinition) 之前。
pdfMake.tableLayouts = {
  exampleLayout: {
    hLineWidth: function (i, node) {
      if (i === 0 || i === node.table.body.length) {
        return 0;
      }
      return (i === node.table.headerRows) ? 2 : 1;
    },
    vLineWidth: function (i) {
      return 0;
    },
    hLineColor: function (i) {
      return i === 1 ? 'black' : '#aaa';
    },
    paddingLeft: function (i) {
      return i === 0 ? 0 : 8;
    },
    paddingRight: function (i, node) {
      return (i === node.table.widths.length - 1) ? 0 : 8;
    }
  }
};
pdfMake.createPdf(docDefinition).download();

// toc: table of content
var docDefinition = {
  content: [
    {
      toc: {
        // id: 'mainToc'  // optional
        title: {text: 'INDEX', style: 'header'}
      }
    },
    {
      text: 'This is a header',
      style: 'header',
      tocItem: true, // or tocItem: 'mainToc' if is used id in toc
      // or tocItem: ['mainToc', 'subToc'] for multiple tocs
    }
  ]
}
```

## 清单

```js
var docDefinition = {
  content: [
    'Bulleted list example:',
    {
      // 无序
      ul: [
        'Item 1',
        'Item 2',
        'Item 3',
        { text: 'Item 4', bold: true },
      ]
    },
    'Numbered list example:',
    {
      // 有序
      ol: [
        'Item 1',
        'Item 2',
        'Item 3'
      ]
    }
  ]
};
```

## 页头/页尾

```js
// 静态
var docDefinition = {
  header: 'simple text',

  footer: {
    columns: [
      'Left part',
      { text: 'Right part', alignment: 'right' }
    ]
  },

  content: (...)
};
// 动态
var docDefinition = {
  footer: function(currentPage, pageCount) { return currentPage.toString() + ' of ' + pageCount; },
  header: function(currentPage, pageCount, pageSize) {
    // 可以返回任何对象
    return [
      { text: 'simple text', alignment: (currentPage % 2) ? 'left' : 'right' },
      { canvas: [ { type: 'rect', x: 170, y: 32, w: pageSize.width - 170, h: 40 } ] }
    ]
  },
  (...)
};
```

## 背景

```js
var docDefinition = {
  background: 'simple text',

  content: (...)
};
// 也可以是个函数、表、图片……
var docDefinition = {
  background: function(currentPage) {
    return 'simple text on page ' + currentPage
  },

  content: (...)
};
```

## 留边

```js
// margin: [left, top, right, bottom]
{ text: 'sample', margin: [ 5, 2, 10, 20 ] },

// margin: [horizontal, vertical]
{ text: 'another text', margin: [5, 2] },

// margin: equalLeftTopRightBottom
{ text: 'last one', margin: 5 }
```

## 堆栈

```js
// 把整篇文档做为一个堆栈来整理这种嵌套结构
var docDefinition = {
  content: [
    'paragraph 1',
    'paragraph 2',
    {
      columns: [
        'first column is a simple text',
        [
          // second column consists of paragraphs
          'paragraph A',
          'paragraph B',
          'these paragraphs will be rendered one below another inside the column'
        ]
      ]
    }
  ]
};
// 数组实际是堆栈 { stack: [] } 的简化版， 数组无法定义样式，而 stack 可以定义样式。
var docDefinition = {
  content: [
    'paragraph 1',
    'paragraph 2',
    {
      columns: [
        'first column is a simple text',
        {
          stack: [
            // second column consists of paragraphs
            'paragraph A',
            'paragraph B',
            'these paragraphs will be rendered one below another inside the column'
          ],
          fontSize: 15
        }
      ]
    }
  ]
};
```

## 图像

```js
var docDefinition = {
  content: [
    {
      // 最常用的是使用 dataURI 的图像
      // 如果不定义 宽/高/自适应 width/height/fit，图像将是原始尺寸
      image: 'data:image/jpeg;base64,...encodedContent...'
    },
    {
      // 如果规定有宽度，图像将按比例缩放
      image: 'data:image/jpeg;base64,...encodedContent...',
      width: 150
    },
    {
      // 如果规定有宽度和高度，图像将被拉伸
      image: 'data:image/jpeg;base64,...encodedContent...',
      width: 150,
      height: 150
    },
    {
      // 也可以规定一个矩形大小，图像在其中自适应
      image: 'data:image/jpeg;base64,...encodedContent...',
      fit: [100, 100]
    },
    {
      // 如果图像需要重用，可以先放入字典中，然后按名字引用
      image: 'mySuperImage'
    },
    {
      // 在虚拟文件系统或 NodeJS 中可以使用文件路径/文件名来引入图片
      image: 'myImageDictionary/image1.jpg'
    }
  ],
  images: {
    mySuperImage: 'data:image/jpeg;base64,...content...'
  }
};
```

## 链接

```js
{text: 'google', link: 'http://google.com'}
{text:'Go to page 2', linkToPage: 2}
```

## 页面尺寸/方向/边距

```js
var docDefinition = {

  // 页面尺寸： 传入以下纸张类型字符串 或 对象如：{ width: number, height: number }
  // '4A0', '2A0', 'A0', 'A1', 'A2', 'A3', 'A4', 'A5', 'A6', 'A7', 'A8', 'A9', 'A10',
  // 'B0', 'B1', 'B2', 'B3', 'B4', 'B5', 'B6', 'B7', 'B8', 'B9', 'B10',
  // 'C0', 'C1', 'C2', 'C3', 'C4', 'C5', 'C6', 'C7', 'C8', 'C9', 'C10',
  // 'RA0', 'RA1', 'RA2', 'RA3', 'RA4',
  // 'SRA0', 'SRA1', 'SRA2', 'SRA3', 'SRA4',
  // 'EXECUTIVE', 'FOLIO', 'LEGAL', 'LETTER', 'TABLOID'
  pageSize: 'A5',

  // 页面方向：默认 portrait（肖像）竖向， 可以调整为 landscape （风景）横向
  pageOrientation: 'landscape',

  // 页边距： [left, top, right, bottom] 或 [horizontal, vertical] 
  pageMargins: [ 40, 60, 40, 60 ],
};
```

## 分页

```js
'You can also fit the image inside a rectangle',
{
  image: 'fonts/sampleImage.jpg',
  fit: [100, 100],
  pageBreak: 'after'
},

{ text: 'noBorders:', fontSize: 14, bold: true, pageBreak: 'before', margin: [0, 0, 0, 8] },
```

```js
// Dynamically control page breaks, for instance to avoid orphan childs
// Can be specify a pageBreakBefore function, which can determine if a page break should be inserted before the page break. To implement a 'no orphan child' rule, this could like like this:
var dd = {
    content: [
       {text: '1 Headline', headlineLevel: 1},
       'Some long text of variable length ...',
       {text: '2 Headline', headlineLevel: 1},
       'Some long text of variable length ...',
       {text: '3 Headline', headlineLevel: 1},
       'Some long text of variable length ...',
    ],
  pageBreakBefore: function(currentNode, followingNodesOnPage, nodesOnNextPage, previousNodesOnPage) {
     return currentNode.headlineLevel === 1 && followingNodesOnPage.length === 0;
  }
}

// If pageBreakBefore returns true, a page break will be added before the currentNode. Current node has the following information attached:
{
   id: '<as specified in doc definition>', 
   headlineLevel: '<as specified in doc definition>',
   text: '<as specified in doc definition>', 
   ul: '<as specified in doc definition>', 
   ol: '<as specified in doc definition>', 
   table: '<as specified in doc definition>', 
   image: '<as specified in doc definition>', 
   qr: '<as specified in doc definition>', 
   canvas: '<as specified in doc definition>', 
   columns: '<as specified in doc definition>', 
   style: '<as specified in doc definition>', 
   pageOrientation '<as specified in doc definition>',
   pageNumbers: [2, 3], // The pages this element is visible on (e.g. multi-line text could be on more than one page)
   pages: 6, // the total number of pages of this document
   stack: false, // if this is an element which encapsulates multiple sub-objects
   startPosition: {
     pageNumber: 2, // the page this node starts on
     pageOrientation: 'landscape', // the orientation of this page
     left: 60, // the left position
     right: 60, // the right position
     verticalRatio: 0.2, // the ratio of space used vertically in this document (excluding margins)
     horizontalRatio: 0.0  // the ratio of space used horizontally in this document (excluding margins)
   }
}
```

## 元数据

```javascript
var docDefinition = {
  info: {
	title: 'awesome Document',
	author: 'john doe',
	subject: 'subject of document',
	keywords: 'keywords for document',
  },
  content:  'This is an sample PDF printed with pdfMake'
}

// title - the title of the document
// author - the name of the author
// subject - the subject of the document
// keywords - keywords associated with the document
// creator - the creator of the document (default is 'pdfmake')
// producer - the producer of the document (default is 'pdfmake')
// creationDate - the date the document was created (added automatically by pdfmake)
// modDate - the date the document was last modified
// trapped - the trapped flag in a PDF document indicates whether the document has been "trapped"
```

## 文档压缩

```js
// 默认压缩文档
var docDefinition = {
  compress: false,
  content: (...)
};
```

## 客户端使用中自定义字体

1. pdfmake 通过执行 `gulp buildFonts` 命令将 `examples/font` 下的所有字体和图片 以键值对的形式重新嵌入到 `build/vfs_fonts.js` 文件中。
    - 可以通过修改 `gulpfile.js` 调整 gulp buildFonts 任务中定义的默认文件路径
    - 重新生成 `vfs_font.js` 后，只需要引用该文件即可，不需要再引用实际的字体
2. 设置字体，并在 pdf 文档定义中使用字体样式

```js
pdfMake.fonts = {
   // default font 
   Roboto: {
    normal: 'Roboto-Regular.ttf',
    bold: 'Roboto-Medium.ttf',
    italics: 'Roboto-Italic.ttf',
    bolditalics: 'Roboto-MediumItalic.ttf'
   },
   // custom font
   // Each font-family defines 4 properties: normal, bold, italics and bolditalics
   // 4 properties can all point to the same font file
   yourFontName: {
     normal: 'fontFile.ttf',
     bold: 'fontFile2.ttf',
     italics: 'fontFile3.ttf',
     bolditalics: 'fontFile4.ttf'
   },
   // example of usage fonts in collection
   PingFangSC: {
     normal: ['pingfang.ttc', 'PingFangSC-Regular'],
     bold: ['pingfang.ttc', 'PingFangSC-Semibold'],
   }
}

var docDefinition = {
  content: (...),
  defaultStyle: {
    font: 'yourFontName'
  }
}

```

[express]: https://github.com/bpampuch/pdfmake/blob/master/dev-playground/server.js