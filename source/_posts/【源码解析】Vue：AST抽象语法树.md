---
title: 【源码解析】Vue：AST抽象语法树
date: 2021-02-01 23:29:56
categories: 前端
tags:
	- Vue
	- 源码解析
	- AST语法树
	- 模板解析
cover_picture: /images/vue.png
---



# 基础版

```javascript
var templateStr = "<div><p>aaa</p><ul><li>a</li><li>b</li><li>c</li></ul></div>";
parseHtml(templateStr)

function parseHtml(templateStr) {
    // 指针
    var index = 0;
    var stackTag = [];
    var stackContent = [{'children': []}];
    var rest = templateStr;
    var startRegExp = /^\<([a-z]+[1-6]?)\>/;
    var endRegExp = /^\<\/([a-z]+[1-6]?)\>/;
    var wordRegExp = /^([^\<]+)\<\/[a-z]+[1-6]?\>/;
    while (index < templateStr.length - 1) {
        rest = templateStr.substring(index)
        // 识别开始字符是不是开始标签
        if (startRegExp.test(rest)) {
            // 获取开始标记
            let tag = rest.match(startRegExp)[1];
            // 将开始标记推入栈中
            stackTag.push(tag);
            // 将空数组推入栈中
            stackContent.push({'tag': tag, 'children': []});
            index += tag.length + 2;
        } else if (endRegExp.test(rest)) {
            // 获取结束标记
            let tag = rest.match(endRegExp)[1];
            let pop_tag = stackTag.pop();
            // 判断跟栈顶的标记是不是同一个
            if (tag == pop_tag) {
                let pop_arr = stackContent.pop();
                if (stackContent.length > 0) {
                    stackContent[stackContent.length - 1].children.push(pop_arr);
                }
            } else {
                throw new Error(stackTag[stackTag.length - 1] + "没有闭合")
            }
            index += tag.length + 3;
        } else if (wordRegExp.test(rest)) {
            let word = rest.match(wordRegExp)[1];
            if (!/^\s+$/.test(word)) {
                stackContent[stackContent.length - 1].children.push({'text': word, 'type': 3})
            }
            index += word.length;
        } else {
            index++;
        }
    }
    console.log(stackContent[0].children[0])
}
```





# 进阶版

## 1）修改检测开始标记的正则

```javascript
var startRegExp = /^\<([a-z]+[1-6]?)(\s[^\<]+)?\>/;
// 添加检测attr
```

## 2）检测到开始标记时，取出识别到的attr内容，指针后移

```javascript
let attrString = rest.match(startRegExp)[2];
index += tag.length + 2 + attrString.length;
```

## 3）格式化attrString

```javascript
function parseAttrString(attrString) {
    if(attrString == undefined)return [];
    var isYinhao = false;
    var point = 0;
    var result = [];
    for (var i = 0;i<attrString.length;i++){
        let char = attrString[i];
        if(char == '"'){
            isYinhao = !isYinhao;
        }else if (char == ' ' && !isYinhao){
            if(!/^\s*$/.test(attrString.substring(point,i))) {
                result.push(attrString.substring(point,i).trim());
                point = i;
            }
        }
    }
    result.push(attrString.substring(point).trim())

    result = result.map(item => {
        // 根据=拆分
        let o = item.match(/^(.+)="(.+)"$/);
        return {
            name: o[1],
            value: o[2]
        }
    })
    return result
}
```

## 4）添加attrs属性

```javascript
stackContent.push({'tag': tag, 'children': [],'attrs':parseAttrString(attrString)});
```



**完整代码**

```javascript
var templateStr = '<div><p id="div" class="box p">aaa</p><ul><li>a</li><li>b</li><li>c</li></ul></div>';
parseHtml(templateStr)

function parseHtml(templateStr) {
    // 指针
    var index = 0;
    var stackTag = [];
    var stackContent = [{'children': []}];
    var rest = templateStr;
    var startRegExp = /^\<([a-z]+[1-6]?)(\s[^\<]+)?\>/;
    var endRegExp = /^\<\/([a-z]+[1-6]?)\>/;
    var wordRegExp = /^([^\<]+)\<\/[a-z]+[1-6]?\>/;
    while (index < templateStr.length - 1) {
        rest = templateStr.substring(index)
        // 识别开始字符是不是开始标签
        if (startRegExp.test(rest)) {
            // 获取开始标记
            let tag = rest.match(startRegExp)[1];
            let attrString = rest.match(startRegExp)[2];
            // 将开始标记推入栈中
            stackTag.push(tag);
            // 将空数组推入栈中
            stackContent.push({'tag': tag, 'children': [],'attrs':parseAttrString(attrString)});
            index += tag.length + 2 + (attrString != null?attrString.length:0);
        } else if (endRegExp.test(rest)) {
            // 获取结束标记
            let tag = rest.match(endRegExp)[1];
            let pop_tag = stackTag.pop();
            // 判断跟栈顶的标记是不是同一个
            if (tag == pop_tag) {
                let pop_arr = stackContent.pop();
                if (stackContent.length > 0) {
                    stackContent[stackContent.length - 1].children.push(pop_arr);
                }
            } else {
                throw new Error(stackTag[stackTag.length - 1] + "没有闭合")
            }
            index += tag.length + 3;
        } else if (wordRegExp.test(rest)) {
            let word = rest.match(wordRegExp)[1];
            // 判断内容是否为空
            if (!/^\s+$/.test(word)) {
                stackContent[stackContent.length - 1].children.push({'text': word, 'type': 3})
            }
            index += word.length;
        } else {
            index++;
        }
    }
    console.log(stackContent[0].children[0])
}

function parseAttrString(attrString) {
    if(attrString == undefined)return [];
    var isYinhao = false;
    var point = 0;
    var result = [];
    for (var i = 0;i<attrString.length;i++){
        let char = attrString[i];
        if(char == '"'){
            isYinhao = !isYinhao;
        }else if (char == ' ' && !isYinhao){
            if(!/^\s*$/.test(attrString.substring(point,i))) {
                result.push(attrString.substring(point,i).trim());
                point = i;
            }
        }
    }
    result.push(attrString.substring(point).trim())

    result = result.map(item => {
        // 根据=拆分
        let o = item.match(/^(.+)="(.+)"$/);
        return {
            name: o[1],
            value: o[2]
        }
    })
    return result
}
```
