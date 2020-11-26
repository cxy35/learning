---
title: POI 操作手册
date: 2018-04-23 17:14:24
categories: POI
tags: [POI]
toc: true
---
通过本文认识 POI ，并学习 Excel/Word/... 等类型文档的操作。
<!-- more -->

## 1 POI 简介

POI 是 Apache 下的 Jakata 项目的一个子项目，主要用于提供 java 操作 Microsoft Office 办公套件如 Excel，Word，Powerpoint 等文件的 API 。

微软的 Office 办公软件在企业的日常办公中占据着重要的地位，人们已经非常熟悉 Office 的使用。在我们开发的应用系统中，常常需要将数据导出到 Excel 文件中，或者Word 文件中进行打印。比如移动的话费查询系统中就提供了将话费清单导入到 excel 表格中的功能。这样在 web 应用中，我们在浏览器中看到的数据可以被导出到 Excel 中了。

- Excel文件： xls 格式文件对应 POI API 为 HSSF。xlsx 格式为 office 2007 的文件格式，POI 中对应的 API 为XSSF。
- Word文件：doc 格式文件对应的 POI API 为 HWPF 。 docx 格式为 XWPF 。
- PowerPoint文件：ppt 格式对应的 POI API 为 HSLF 。 pptx 格式为 XSLF 。
- Outlook文件：对应的 API 为 HSMF 。
- Visio文件：对应的 API 为 HDGF 。
- Publisher文件：对应的 API 为 HPBF 。

下载地址：[http://poi.apache.org/download.html](http://poi.apache.org/download.html)

## 2 Excel 操作手册

一个 Excel 文档称为工作簿（worksheet），一个工作簿包含多个工作表（sheet），每个工作表看起来像一张二维表格，由很多行（row）组成，每行由多个单元格组成（cell）。

|POI HSSF API 中的类|Excel 结构|
|:-|:-|
|HSSFWorkbook|工作簿|
|HSSFSheet|工作表|
|HSSFRow|行|
|HSSFCell|单元格|
|HSSFCellStyle|单元格样式|
|HSSFFont|字体|
|HSSFDataFormat|单元格日期格式|
|HSSFHeader|sheet 的页眉|
|HSSFFooter|sheet 的页脚|
|HSSFDateUtil|日期|
|HSSFPrintSetup|打印|
|HSSFErrorConstants|错误信息表|

## 3 Word 操作手册

## 4 PowerPoint 操作手册

## 5 Outlook 操作手册

## 6 Visio 操作手册

## 7 Publisher 操作手册


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)