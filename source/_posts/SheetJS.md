---
title: SheetJS #文章标题
date: 2018-06-19 00:00:00  #写作时间
description:  SheetJS，JS实现Excel中的数据读取和数据导出。 #文章描述
categories: #文章分类
- JavaScript
tags: #文章标签
- Excel
toc: true # 生成目录
author: Juphy
comments:
original:
permalink: #指定链接
---
## 安装
- 直接使用
```
<script lang="javascript" src="xlsx.full.min.js"></script>
```
- 使用npm
```
npm instal xlsx
```
- 老版本浏览器
```
<!-- add the shim first -->
<script type="text/javascript" src="shim.min.js"></script>
<!-- after the shim is referenced, add the library -->
<script type="text/javascript" src="xlsx.full.min.js"></script>
```

## Angular 2+

```
import * as XLSX from 'xlsx';
```
> sheet_to_json

```
/* <input type="file" (change)="onFileChange($event)" multiple="false" /> */

  onFileChange(evt: any) {
    /* wire up file reader */
    const target: DataTransfer = <DataTransfer>(evt.target);
    if (target.files.length !== 1) throw new Error('Cannot use multiple files');
    const reader: FileReader = new FileReader();
    reader.onload = (e: any) => {
      /* read workbook */
      const bstr: string = e.target.result;
      const wb: XLSX.WorkBook = XLSX.read(bstr, {type: 'binary'});

      /* grab first sheet */
      const wsname: string = wb.SheetNames[0];
      const ws: XLSX.WorkSheet = wb.Sheets[wsname];

      /* save data */
      this.data = <AOA>(XLSX.utils.sheet_to_json(ws, {header: 1}));
    };
    reader.readAsBinaryString(target.files[0]);
  }

```

> 结合Ant Design使用

```
/*
<nz-upload [nzBeforeUpload]="beforeUpload">
   <button nz-button>导入文件</button>
</nz-upload>
*/

reader: FileReader = new FileReader();

constructor(){
    this.reader.onloadstart=()=>{

    }

    this.reader.onload = (e: any)=>{
        const bstr: string = e.target.result;
        const wb: XLSX.WorkBook = XLSX.read(bstr, {type: 'binary'});
        const sheet = wb.Sheets[wb.SheetNames[0]];
        const json = XLSX.utils.sheet_to_json(sheet);
    }

    this.header.onloadend = () =>{

    }
}

beforeUpload = (file: UploadFile): boolean => {
  this.reader.readAsBinaryString(this.file);
  return false;
}

```
> json_to_sheet/aoa_to_sheet

```
/* generate worksheet */
const ws: XLSX.WorkSheet = XLSX.utils.json_to_sheet(data);
const ws: XLSX.WorkSheet = XLSX.utils.aoa_to_sheet(data);

/* generate workbook and add the worksheet */
const wb: XLSX.WorkBook = XLSX.utils.book_new();
XLSX.utils.book_append_sheet(wb, ws, 'Sheet1');

/* save to file */
XLSX.writeFile(wb, 'SheetJS.xlsx');

或者使用file-saver中的saveAs

import {saveAs} from 'file-save

const wbout: string = XLSX.write(wb, {bookType: 'xlsx', bookSST: true, type: 'binary'});

const s2ab = (s: any) => {
    const buf = new ArrayBuffer(s.length);
    const view = new Uint8Array(buf);
    for (let i = 0; i !== s.length; ++i) view[i] = s.charCodeAt(i) & 0xFF;
    return buf;
};

saveAs(new Blob([s2ab(wbout)], {type: 'application/octet-stream'}), 'sheetJS.xlsx');

```

> 修改某一单元的数据，合并单元格问题

```
const ws: any = XLSX.utils.json_to_sheet(data);
// 修改填充的数据
ws['A1']={t: 's', v: '需要填写的内容'};
ws['A2']={t: 's', v: '需要填写的内容'};
ws['A3']={t: 's', v: '需要填写的内容'};

// 合并单元格
/*
c: 代表纵向，从0开始
r: 代表横向，从0开始
*/
ws['!merges']=[
    {
    s: {c: 0, r: 0},
    e: {c: 10, r: 0}
    },
    {
    s: {c: 1, r: 0},
    e: {c: 1, r: 10}
    }
]
```
> 单元格

|Key| Description|
|:---:|:---:|
|v|原始值|
|w|格式化文本|
|t|type: b Boolean, e Error, n Number, d Date, s Text, z Stub|

> 多种导出形式

- Importing:

```
aoa_to_sheet converts an array of arrays of JS data to a worksheet.
json_to_sheet converts an array of JS objects to a worksheet.
table_to_sheet converts a DOM TABLE element to a worksheet.
sheet_add_aoa adds an array of arrays of JS data to an existing worksheet.
sheet_add_json adds an array of JS objects to an existing worksheet.
```
- Exporting:

```
sheet_to_json converts a worksheet object to an array of JSON objects.
sheet_to_csv generates delimiter-separated-values output.
sheet_to_txt generates UTF16 formatted text.
sheet_to_html generates HTML output.
sheet_to_formulae generates a list of the formulae (with value fallbacks).
```
- Cell and cell address manipulation:

```
format_cell generates the text value for a cell (using number formats).
encode_row / decode_row converts between 0-indexed rows and 1-indexed rows.
encode_col / decode_col converts between 0-indexed columns and column names.
encode_cell / decode_cell converts cell addresses.
encode_range / decode_range converts cell ranges.
```

*其他详细使用，请查看[SheetJS](https://github.com/SheetJS/js-xlsx)*


