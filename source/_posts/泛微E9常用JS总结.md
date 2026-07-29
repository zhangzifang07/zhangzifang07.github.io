---
title: 泛微E9常用JS总结
excerpt: 泛微E9流程、建模、ecode等常用JS代码总结，包含字段操作、明细表、事件绑定、提交校验、弹框、接口请求等示例
date: 2026-07-29 11:40:48
permalink: /2026/07/29/e9-common-js-summary/
tags:
  - 泛微E9
  - JavaScript
  - 前端
categories:
  - 泛微开发
---

本文整理泛微E9中常用的前端JS代码，按照流程表单、建模和ecode通用前端能力分类。

示例中的字段ID、明细表名称、接口地址和业务函数需要根据实际场景替换。E9 ecode运行环境已提供 `React`、`antd`、`jQuery` 等全局对象，代码块中无需使用 `import` 引入。

# 一、流程表单（WfForm）

## 1. 获取字段ID

主表字段只需要传字段名称；明细字段还需要传所属明细表名称。

```js
var fylx1id = WfForm.convertFieldNameToId("fylx1");
var fieldid = WfForm.convertFieldNameToId("materialCode", "detail_4");
```

## 2. 获取字段值

获取明细字段值时，需要将字段ID和行索引拼接起来。

```js
var fieldvalue = WfForm.getFieldValue(
    WfForm.convertFieldNameToId("zrr") + "_" + rowIndex
);
```

## 3. 遍历明细表

```js
var fieldid = WfForm.convertFieldNameToId("materialCode", "detail_4");
var rowArr = WfForm.getDetailAllRowIndexStr("detail_4").split(",");

for (var i = 0; i < rowArr.length; i++) {
    var rowIndex = rowArr[i];
    if (rowIndex !== "") {
        var fieldMark = fieldid + "_" + rowIndex;
        WfForm.setTextFieldEmptyShowContent(
            fieldMark,
            "多个编码请用/进行分隔"
        );
    }
}
```

`setTextFieldEmptyShowContent` 用于设置文本字段为空时显示的灰色提示内容。

## 4. 绑定主表字段变化事件

```js
WfForm.bindFieldChangeEvent(ywleixinid, function (obj, id, value) {
    console.log("业务类型发生变化：" + value);
    initdetail();
    // getWorkflowData(wlbmid);
    initmain();
});
```

## 5. 强制触发字段联动

```js
jQuery(document).ready(function () {
    setTimeout(function () {
        var fymcvalu_dt = WfForm.getFieldValue(fymcbmid + "_0");
        if (fymcvalu_dt != "") {
            WfForm.triggerFieldAllLinkage(fymcbmid + "_0");
        }
    }, 1000);
});
```

## 6. 控制明细表列的显示和隐藏

```js
function handleHide(value) {
    if (value == 0) {
        // 隐藏第1、2列，显示第5列
        jQuery(".detail_column1").addClass("detail_hide_col");
        jQuery(".detail_column2").addClass("detail_hide_col");
        jQuery(".detail_column5").removeClass("detail_hide_col");
    } else if (value == 1) {
        // 隐藏第5列，显示第1、2列
        jQuery(".detail_column5").addClass("detail_hide_col");
        jQuery(".detail_column1").removeClass("detail_hide_col");
        jQuery(".detail_column2").removeClass("detail_hide_col");
    } else {
        // 显示全部列
        jQuery(".detail_column1").removeClass("detail_hide_col");
        jQuery(".detail_column2").removeClass("detail_hide_col");
        jQuery(".detail_column5").removeClass("detail_hide_col");
    }
}

// 表单加载后先执行一次
var value = WfForm.getFieldValue("field10009");
handleHide(value);

// 条件字段变化时重新执行
WfForm.bindFieldChangeEvent("field10009", function (obj, id, value) {
    handleHide(value);
});

// 新增明细行时重新执行，数字1代表明细表序号
WfForm.registerAction(WfForm.ACTION_ADDROW + "1", function (index) {
    var value = WfForm.getFieldValue("field10009");
    handleHide(value);
});

// 切换明细表分页时重新执行
WfForm.registerAction(WfForm.ACTION_SWITCHDETAILPAGING, function (groupid) {
    var value = WfForm.getFieldValue("field10009");
    handleHide(value);
});
```

## 7. 绑定明细字段变化事件

```js
WfForm.bindDetailFieldChangeEvent(
    "field27583,field27584",
    function (id, rowIndex, value) {
        console.log(
            "WfForm.bindDetailFieldChangeEvent--",
            id,
            rowIndex,
            value
        );
    }
);
```

## 8. 多行文本框自动拉伸

```js
document.getElementById("field6990").addEventListener("keyup", function (e) {
    if (e.keyCode === 8) {
        this.style.height = "inherit";
        this.style.height = `${this.scrollHeight}px`;
    } else {
        this.style.height = `${this.scrollHeight}px`;
    }
    this.style.overflow = "hidden";
});
```

## 9. 新增明细行并赋值

```js
WfForm.addDetailRow("detail_3", {
    field14989: {
        value: "432",
        specialobj: [
            { id: "432", name: "库存商品" }
        ]
    }, // 科目名称，浏览框
    field14988: { value: "你好哇" }, // 摘要
    field14991: { value: "币别" }, // 币别
    field14992: { value: "汇率类型" }, // 汇率类型
    field14993: { value: "0" }, // 余额方向
    field14994: { value: "100" } // 借方金额
});
```

浏览框除 `value` 外，还需要通过 `specialobj` 提供显示名称。

## 10. 提交前校验并提示

```js
WfForm.registerCheckEvent(
    WfForm.OPER_SAVE + "," + WfForm.OPER_SUBMIT,
    function (callback) {
        var businessType = WfForm.getFieldValue("field9477");

        if (businessType == "0") {
            var dt13 = WfForm.getDetailAllRowIndexStr("detail_13");
            var arrList = dt13.split(",");
            var sfyczid = WfForm.convertFieldNameToId("sfycz", "detail_13");

            for (var i = 0; i < arrList.length; i++) {
                var xymxidValue = WfForm.getFieldValue(
                    sfyczid + "_" + arrList[i]
                );

                if (xymxidValue != "") {
                    antd.Modal.warning({
                        title: "系统提示",
                        content: "您输入的物料描述已经存在，请重新输入",
                        okText: "确定"
                    });
                    return;
                }
            }
        }

        // 校验通过后必须调用callback，否则保存或提交不会继续
        callback();
    }
);
```

## 11. 流程确认框

```js
WfForm.showConfirm("确认删除吗？", function () {
    alert("删除成功");
});
```

## 12. 调用ESB接口并回写字段

```js
var params = {
    wlbm: filevalue
};
var jsonStr = JSON.stringify(params);

jQuery.ajax({
    url: "/api/esb/oa/execute?&eventkey=checkMaterials&params=" + jsonStr,
    type: "POST",
    data: "",
    success: function (ret) {
        try {
            var retObj = JSON.parse(ret);
            if (retObj.data && retObj.data.code === "200") {
                var answer = retObj.data.answer;
                WfForm.changeFieldValue(
                    WfForm.convertFieldNameToId("jxycswb"),
                    { value: answer }
                );
                WfForm.changeFieldValue(
                    WfForm.convertFieldNameToId("jxycsdhwb"),
                    { value: answer }
                );
            }
        } catch (e) {
            console.error("解析返回数据时出错：", e);
        }
    },
    error: function (jqXHR, textStatus, errorThrown) {
        console.error("AJAX请求失败：", textStatus, errorThrown);
    },
    complete: function () {
        // 请求完成后恢复按钮状态
        checkBtn.disabled = false;
        checkBtn.value = "测试";
        isProcessing = false;
    }
});
```

## 13. 根据字段值修改页面样式

```js
var sfcb_flag = WfForm.getFieldValue("field7134");

if (sfcb_flag == 0) {
    updateElementStyle("#sfcb > span > span.child-item.wdb", "red", "bold");
} else {
    updateElementStyle("#sfcb > span > span.child-item.wdb", "black", "normal");
}

WfForm.bindFieldChangeEvent("field7134", function (id, rowIndex, value) {
    var color;
    var fontWeight;

    if (value == 0) {
        color = "red";
        fontWeight = "bold";
    } else {
        color = "black";
        fontWeight = "normal";
    }

    setTimeout(function () {
        updateElementStyle(
            "#sfcb > span > span.child-item.wdb",
            color,
            fontWeight
        );
    }, 10);
});

function updateElementStyle(selector, color, fontWeight) {
    try {
        var element = document.querySelector(selector);
        if (element) {
            element.style.color = color;
            element.style.fontWeight = fontWeight;
        } else {
            console.warn("元素未找到: " + selector);
        }
    } catch (error) {
        console.error("样式更新失败: ", error);
    }
}
```

## 14. Ant Design警告框

```js
antd.Modal.warning({
    title: "系统提示",
    content: "123",
    okText: "确定",
    closable: true,
    onOk: function () {
        console.log(1);
        return Promise.resolve();
    }
});
```

## 15. 带取消按钮的确认框

```js
antd.Modal.confirm({
    title: "系统提示",
    content: "请注意,第1行该科目预算总额不足",
    okText: "确定",
    cancelText: "取消",
    closable: true,
    onOk: function () {
        console.log(1);
        return Promise.resolve();
    },
    onCancel: function () {
        console.log(2);
        return Promise.resolve();
    }
});
```

## 16. 显示带HTML格式的内容

只应把可信内容传给 `dangerouslySetInnerHTML`，不要直接传入未经处理的用户输入。

```js
var content = "第一行<br>第二行";
var modalInstance = antd.Modal.confirm({
    title: "系统提示",
    content: React.createElement("div", {
        dangerouslySetInnerHTML: { __html: content }
    }),
    okText: "确定",
    cancelText: "取消",
    width: "80%",
    onOk: function () {
        callback();
        modalInstance = null;
    },
    onCancel: function () {
        modalInstance = null;
    },
    onClose: function () {
        modalInstance = null;
    }
});
```

## 17. 使用jQuery添加按钮

```js
jQuery(".butn").html(
    "<input type='button' style='color:white' " +
    "class='ant-btn ant-btn-primary' value='测试' " +
    "onclick='checkMaterials()' id='checkBtn'>"
);
```


# 二、建模（ModeForm / ModeList）

## 1. 建模表单新增明细行

```js
ModeForm.addDetailRow("detail_1", {
    field11033: { value: "2025-02-6" }
});
```

## 2. 获取列表勾选数据并调用系统接口

```js
function closeDsh() {
    var ids = ModeList.getCheckedID();

    jQuery.ajax({
        type: "GET",
        url: "/api/caigou/order/closeDsh",
        dataType: "json",
        data: { ids: ids },
        cache: false,
        async: false,
        success: function (data, textStatus, jqXHR) {
            if (data.result) {
                ModeList.reloadTableAll();
            } else {
                ModeList.showMessage(data.message, 1, 5);
            }
        },
        error: function (XMLHttpRequest, textStatus, errorThrown) {
            ModeList.showModalMsg("系统提示", "系统异常，请稍后再试", 1);
        }
    });
}
```

## 3. 建模页面扩展按钮二次确认

在建模页面扩展按钮的确认JS中配置：

```text
javascript:getConfirmMessage()
```

对应函数：

```js
function getConfirmMessage() {
    var tableData = ModeList.getTableDatas()[0];
    return tableData.smjd == 1007
        ? (ModeForm.showMessage(
            "注册状态不允许发起变更，请先发起认证注册流程"
        ), false)
        : "是否确认";
}
```

# 三、ecode / 通用前端

## 1. 拦截接口
``` js
let interceptEnable = true;
ecodeSDK.rewriteApiDataQueueSet({
    fn: (url, params, data) => {
        // console.log(497)
        if (!interceptEnable) return data; // 限制条件
        if (window.location.hash.indexOf('#/budget/bankEnterpriseReport/payInfoReview') > -1) {
            // console.log(111)
            if (url == '/api/fna/bankEnterpriseConnect/getTransferReviewAdvanceSearch') {
                // console.log(222)
                // let options = data.conditions[0].items[2].options;
                // let temp = options[0];
                // options[0] = options[1];
                // options[1] = temp;
                data.conditions[0].items[2].value = '1';
                // console.log(data.conditions[0].items[2])
            }



        }
        return data;
    },
    desc: '拦截接口参数，同一个接口会在多个页面请求，需注意路径的判断'
});
```
## 2. 复写组件
```js
ecodeSDK.overwritePropsFnQueueMapSet('WeaReqTop', { //组件名
    fn: (newProps) => { //newProps代表组件参数
        //进行位置判断
        if (!ecodeSDK.checkLPath('/spa/workflow/static4form/index.html#/main/workflow/req')) return;
        var workflowId = wfform.getBaseInfo().workflowid;
        // console.log('workflow判断',workflowId!=152)
        if (workflowId && workflowId == 153) {

            console.log("开始了123")

            jQuery("body > div:nth-child(42) > div > div.ant-modal-wrap > div > div.ant-modal-content > div > div > div.ant-confirm-body > div.ant-confirm-header > span").wait(function () {
                console.log("找到了")
                jQuery("body > div:nth-child(42) > div > div.ant-modal-wrap > div > div.ant-modal-content > div > div > div.ant-confirm-btns > button").hide(); //隐藏确定按钮
            });
        }


    },
    order: 1, //排序字段，如果存在同一个页面复写了同一个组件，控制顺序时使用
    desc: '在这里写此复写的作用，在调试的时候方便查找'
});
```


## 3. 拦截入参
```js
let interceptEnable2 = true;
ecodeSDK.rewriteApiParamsQueueSet({
    fn: (url, method, params) => {
        if (interceptEnable2 && url.indexOf('/api/fna/report/implementationReport') >= 0) {
            // console.log(236)
            params.orgId = 1;
            interceptEnable2 = false;
            // console.log(params)
            // params = { ...params }
        }

        return {
            url: url, // 接口路径
            method: method, // 请求类型
            params: params // 	请求参数
        }
    },
    desc: '复写移动端接口传参'
});
```