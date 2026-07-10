---
title: E10JS总结
excerpt: E10流程与建模表单的JS-SDK使用总结，包括获取字段ID、读写表单值、明细表操作等
date: 2026-06-29 16:00:00
permalink: /2026/06/29/e10-js-summary/
tags:
  - E10
  - JavaScript
  - 前端
categories:
  - 泛微开发
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

## 3、请求自定义动作流

```js
/* js请求动作流 */
var formSdk = window.WeFormSDK.getWeFormInstance();
var request = window.weappUtils.request;

var ESB_FLOW_ID = '862964717038211072';
var pkid = formSdk.getFieldValue(formSdk.convertFieldNameToId('fid'));
request({
    url: '/api/esb/server/event/triggerActionFlow',
    method: 'POST',
    data: {
        customParams: {
            mainTable:
            {
                username: '0100112',
                pkid: pkid
            }

        },
        moduleSource: 'ecode',
        esbFlowId: ESB_FLOW_ID
    }
}).then(function (res) {
    console.log('kingdee sso res:', res);

    if (res.resultCode == '200' || res.resultCode == 200 || res.code == 200 || res.code == '200') {
        var ssoUrl =
            res.actionData?.responseData?.customData?.mainTable?.ssoUrl ||
            res.data?.ssoUrl ||
            res.result?.ssoUrl ||
            res.ssoUrl;

        if (ssoUrl) {
            window.WeFormSDK.showMessage('金蝶单点链接生成成功', 3, 2);
            window.open(ssoUrl, '_blank');
        } else {
            window.WeFormSDK.showMessage('动作流成功，但未取到 ssoUrl，请看控制台返回', 2, 5);
        }
    } else {
        window.WeFormSDK.showMessage('动作流调用失败：' + (res.resultMsg || ''), 2, 5);
    }
});
```