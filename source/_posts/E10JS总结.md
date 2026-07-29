---
title: 泛微E10 JS总结
excerpt: 泛微E10流程表单与ecode常用JS总结，包括表单实例、字段取值、明细表遍历、保存前校验和自定义动作流调用
date: 2026-06-29 16:00:00
permalink: /2026/06/29/e10-js-summary/
tags:
  - 泛微E10
  - JavaScript
  - 前端
categories:
  - 泛微开发
---

本文整理泛微E10中常用的前端JS代码，按照流程表单、建模表单和ecode集成分类。

示例中的字段名称、明细表名称、动作流ID和业务参数需要根据实际场景替换。

# 一、流程

## 1. 获取表单实例

流程表单相关接口通过 `window.WeFormSDK` 获取实例后调用。

```js
var formSdk = window.WeFormSDK.getWeFormInstance();
```

## 2. 字段名称转字段ID

```js
// 获取主表字段ID：测试开始时间
const cskssj_id = formSdk.convertFieldNameToId("cskssj");
```

## 3. 获取浏览字段数据对象

`specialObj` 的常见结构为 `[{ id: "111", name: "研发部-张三" }]`。

```js
// yfb_id为已经转换好的浏览字段ID
const yfb_fieldObj = formSdk.getFieldObj(yfb_id).specialObj;
```

## 4. 查看主表浏览字段信息

下面的代码同时显示字段原始值、浏览选项ID和显示名称，适合调试浏览字段。

```js
var formSdk = window.WeFormSDK.getWeFormInstance();
var fieldMark = formSdk.convertFieldNameToId("ccbthj");

alert(
    "value=" + formSdk.getFieldValue(fieldMark) +
    "\noptionId=" + formSdk.getBrowserOptionId(fieldMark, ",") +
    "\nshowName=" + formSdk.getBrowserShowName(fieldMark, ",")
);
```

## 5. 遍历明细表浏览字段

先获取明细表的全部行ID，再拼接明细字段ID和行ID读取每一行的数据。

```js
var formSdk = window.WeFormSDK.getWeFormInstance();
var detailMark = formSdk.convertFieldNameToId("fyxx_879809036117371");
var fieldMark = formSdk.convertFieldNameToId("fylx", detailMark);
var rowIds = formSdk.getDetailAllRowIndexStr(detailMark).split(",");
var msg = "";

rowIds.forEach(function (rowId, index) {
    var realFieldMark = fieldMark + "_" + rowId;
    msg += "第" + (index + 1) + "行" +
        "\nvalue=" + formSdk.getFieldValue(realFieldMark) +
        "\noptionId=" + formSdk.getBrowserOptionId(realFieldMark, ",") +
        "\nshowName=" + formSdk.getBrowserShowName(realFieldMark, ",") +
        "\n\n";
});

prompt("复制内容：", msg || "明细表暂无数据");
alert(msg || "明细表暂无数据");
```

## 6. 保存前校验相邻明细行时间

保存表单前，依次比较当前行的预计开始时间和上一行的预计结束时间：

- 当前行开始时间必须晚于上一行结束时间。
- 校验通过时调用 `successFn()` 放行。
- 校验失败时调用 `failFn({ msg })` 阻止保存并显示错误信息。

```js
var formSdk = window.WeFormSDK.getWeFormInstance();

const checktime = (successFn, failFn) => {
    var detailMark = formSdk.convertFieldNameToId("ygclsqd_mxb1");
    var startDateMark = formSdk.convertFieldNameToId("yjksrqx");
    var startTimeMark = formSdk.convertFieldNameToId("yjkssjx");
    var endDateMark = formSdk.convertFieldNameToId("yjjsrqx");
    var endTimeMark = formSdk.convertFieldNameToId("yjjssjx");

    var rowIdStr = formSdk.getDetailAllRowIndexStr(detailMark) || "";
    var rowIdarr = rowIdStr.split(",").filter(rowId => rowId);

    let errorMsg = "";

    for (var i = 1; i < rowIdarr.length; i++) {
        var previousRowId = rowIdarr[i - 1];
        var currentRowId = rowIdarr[i];

        var previousEndDate = formSdk.getFieldValue(
            endDateMark + "_" + previousRowId
        );
        var previousEndTime = formSdk.getFieldValue(
            endTimeMark + "_" + previousRowId
        );

        var currentStartDate = formSdk.getFieldValue(
            startDateMark + "_" + currentRowId
        );
        var currentStartTime = formSdk.getFieldValue(
            startTimeMark + "_" + currentRowId
        );

        var previousEndDateTime = new Date(
            `${String(previousEndDate).replace(/-/g, "/")} ${previousEndTime}`
        );

        var currentStartDateTime = new Date(
            `${String(currentStartDate).replace(/-/g, "/")} ${currentStartTime}`
        );

        if (currentStartDateTime <= previousEndDateTime) {
            errorMsg = `第${i + 1}行的预计开始日期时间必须晚于第${i}行的预计结束日期时间`;
            break;
        }
    }

    errorMsg ? failFn({ msg: errorMsg }) : successFn();

    /* 等价的展开写法：
    if (errorMsg) {
        failFn({ msg: errorMsg });
    } else {
        successFn();
    }
    */
};

formSdk.registerCheckEvent(
    window.WeFormSDK.OPER_SAVE,
    (successFn, failFn) => {
        checktime(successFn, failFn);
    }
);
```

# 二、建模

当前文章暂无建模表单JS示例。后续可在此分类补充字段操作、明细表操作和建模列表事件。

# 三、ecode

## 1. 请求自定义动作流

该示例从表单中获取业务主键，调用ESB自定义动作流，并从返回结果中提取单点登录地址。

```js
var formSdk = window.WeFormSDK.getWeFormInstance();
var request = window.weappUtils.request;

var ESB_FLOW_ID = "862964717038211072";
var pkid = formSdk.getFieldValue(
    formSdk.convertFieldNameToId("fid")
);

request({
    url: "/api/esb/server/event/triggerActionFlow",
    method: "POST",
    data: {
        customParams: {
            mainTable: {
                username: "0100112",
                pkid: pkid
            }
        },
        moduleSource: "ecode",
        esbFlowId: ESB_FLOW_ID
    }
}).then(function (res) {
    console.log("kingdee sso res:", res);

    var isSuccess =
        res.resultCode == "200" ||
        res.resultCode == 200 ||
        res.code == 200 ||
        res.code == "200";

    if (isSuccess) {
        var ssoUrl =
            res.actionData?.responseData?.customData?.mainTable?.ssoUrl ||
            res.data?.ssoUrl ||
            res.result?.ssoUrl ||
            res.ssoUrl;

        if (ssoUrl) {
            window.WeFormSDK.showMessage(
                "金蝶单点链接生成成功",
                3,
                2
            );
            window.open(ssoUrl, "_blank");
        } else {
            window.WeFormSDK.showMessage(
                "动作流成功，但未取到 ssoUrl，请看控制台返回",
                2,
                5
            );
        }
    } else {
        window.WeFormSDK.showMessage(
            "动作流调用失败：" + (res.resultMsg || ""),
            2,
            5
        );
    }
});
```

返回值兼容多个常见层级，实际项目中应根据动作流的固定响应结构适当精简。
