---
title: E10JS总结
date: 2026-06-29 16:00:00
permalink: /2026/06/29/e10-js-summary/
tags:
  - E10
  - JavaScript
  - 前端
categories:
  - 技术学习
---

## 1、流程
- 获取流程的字段id
```js	

// 获取主表字段fieldId
const cskssj_id = formSdk.convertFieldNameToId("cskssj");//测试开始时间

//获取浏览字段的obj [{id: "111", name: "研发部-张三"}]
const yfb_fieldObj = formSdk.getFieldObj(yfb_id).specialObj;//研发部数据obj


//控制台输出流程表单的值 主表
var formSdk = window.WeFormSDK.getWeFormInstance();
var fieldMark = formSdk.convertFieldNameToId('ccbthj');
alert('value=' + formSdk.getFieldValue(fieldMark) + '\noptionId=' + formSdk.getBrowserOptionId(fieldMark, ',') + '\nshowName=' + formSdk.getBrowserShowName(fieldMark, ','));

  
//明细表
var formSdk = window.WeFormSDK.getWeFormInstance();
var detailMark = formSdk.convertFieldNameToId('fyxx_879809036117371');
var fieldMark = formSdk.convertFieldNameToId('fylx', detailMark);
var rowIds = formSdk.getDetailAllRowIndexStr(detailMark).split(",");
var msg = '';
rowIds.forEach(function (rowId, index) {
    var realFieldMark = fieldMark + '_' + rowId;
    msg += '第' + (index + 1) + '行' +
        '\nvalue=' + formSdk.getFieldValue(realFieldMark) +
        '\noptionId=' + formSdk.getBrowserOptionId(realFieldMark, ',') +
        '\nshowName=' + formSdk.getBrowserShowName(realFieldMark, ',') +
        '\n\n';
});
prompt('复制内容：', msg || '明细表暂无数据');
alert(msg || '明细表暂无数据');

    
```

## 2、建模