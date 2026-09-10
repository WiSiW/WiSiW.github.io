---
title: '【JavaScript】获取光标位置'
date: 2023-11-05 20:25:46
tags:
  - JavaScript
  - 光标
---

# 原理
DOM中并没有直接获取光标位置的方法，那么我们只能间接来获取光标位置。DOM支持获取光标选中的范围，我们可以以此为切入点，来获取或定位光标的位置。当选取范围起始点和结束点一样时，就是光标插入的位置。

## 

1.anchor(瞄点)：选区起点。
2.focus(焦点)：选区终点。
3.range(范围)：选区范围，包含整个节点或节点的一部分。

## Selection
Selection: Selection对象表示用户选择的文本范围或插入符号的位置。

经过实验发现Selection选取的节点范围都是块级节点。input 和 texteare 并不能作为Selection的节点

Selection对象存在于window对象上，可以通过window.getSelection()获取示例。

属性：

anchorNode: 选区起点的节点。
anchorOffset:选区的起点位置。
focusNode:选区终点的节点。
focusOffset:选区的终点位置。
isCollapsed:起点和终点是否重叠。
rangeCount:选区包含的range数目。
方法

getRangeAt(index):获取指定的选取范围。
addRange(range):将一个范围添加到Selection对象中。
removeRange():移出指定的范围。
removeAllRanges():移出所有range对象。
collapse(parentNode,offset):将光标移动到parentNode节点的offset位置。
collapseToStart():取消当前选区，并把光标定位在原选区的最开始处，如果此时光标所处的位置是可编辑的，且它获得了焦点，则光标会在原地闪烁。
collapseToEnd():取消当前选区，并将光标定位到原选取的最末位。如果此时光标所处的位置是可编辑的，且它获得了焦点，则光标会在原地闪烁。
extend(node,offset):将终点位置移动到node节点的offset位置。
modify(alter,direction,granularity):通过alter方式(move/extend)来改变光标位置，移动方向为direction(left/right)，移动单位为granularity。
containsNode(aNode,aPartlyContained):判断aNode是否包含在Selection中。aPartlyContained为false表示全包含，为true表示只要部分包含即可。
toString():放回当前Selection对象的字符串。

## Range
Range对象表示一个Selection的选择范围，一个Selection可以包含多个Range。

获取对象

document.createRange():创建一个Range。

selection.getRangeAt(index):获取指定的Range。

属性

collapsed:判断起始位置是否重合。
endContaniner:range终点节点。
endOffset:range的终点位置。
startContaniner:ranstartge起点节点。
startOffset:range的起点位置。
commonAncestorContainer:包含起始点的节点。
方法

setStart(startNode,startOffset):设置范围在startNode的起始位置为startOffset。
setEnd(endNode,endOffset):设置范围在endNode的起始位置为endOffset。
selectNode(referenceNode):设置range的节点为referenceNode。
selectNodeContents(referenceNode):设置range的内容为referenceNode。
collapse(toStart):向边界点折叠range，即是设置光标位置，toStart默认为false，表示光标定位在节点末尾。true表示光标定位在节点起点。
cloneContents():克隆一个range的内容片段。
deleteContents():删除range的内容片段。
extractContents():将range的内容从文档树移动到文档片段中。
insertNode(newNode):在range的其实位置插入新的节点。
surroundContents(newNode):将range对象的内容移动到新的节点中。
cloneRange():克隆一个range对象。
detach():释放当前range。

## input/textarea
在html5中，可输入性表单元素（input/textarea）都存在以下属性。不支持IE6/7。

selectionDirection:forward | backward | none,选区方向
selectionEnd:选区终点位置
selectionStart:选区起点位置
setSelectionRange(selectionStart, selectionEnd, [selectionDirection]):设置获取焦点的输入性元素的选区范围。

# 2.获取光标位置

## 2.1 可编辑div获取光标位置

```javascript
//获取当前光标位置
const getCursortPosition = function (element) {
  var caretOffset = 0;
  var doc = element.ownerDocument || element.document;
  var win = doc.defaultView || doc.parentWindow;
  var sel;
  if (typeof win.getSelection != "undefined") {//谷歌、火狐
    sel = win.getSelection();
    if (sel.rangeCount > 0) {//选中的区域
      var range = win.getSelection().getRangeAt(0);
      var preCaretRange = range.cloneRange();//克隆一个选中区域
      preCaretRange.selectNodeContents(element);//设置选中区域的节点内容为当前节点
      preCaretRange.setEnd(range.endContainer, range.endOffset);  //重置选中区域的结束位置
      caretOffset = preCaretRange.toString().length;
    }
  } else if ((sel = doc.selection) && sel.type != "Control") {//IE
    var textRange = sel.createRange();
    var preCaretTextRange = doc.body.createTextRange();
    preCaretTextRange.moveToElementText(element);
    preCaretTextRange.setEndPoint("EndToEnd", textRange);
    caretOffset = preCaretTextRange.text.length;
  }
  return caretOffset;
}
```

获取光标的位置是先通过获取鼠标的选取范围，然后克隆该选取范围，修改克隆范围的结束位置，这样克隆的范围就只剩下起点到结束点的内容，光标之后的内容被截取扔掉了。所以可以通过剩余内容的长度来确定光标位置。之所以要克隆一个选取范围出来，是为了避免修改光标结束位置时影响到原先内容。

## input/textarea获取光标位置

```javascript
//输入框获取光标
const getPosition = function (element) {
    let cursorPos = 0;
    if (document.selection) {//IE
        var selectRange = document.selection.createRange();
        selectRange.moveStart('character', -element.value.length);
        cursorPos = selectRange.text.length;
    } else if (element.selectionStart || element.selectionStart == '0') {
        cursorPos = element.selectionStart;
    }
    return cursorPos;
}
```

# 3.设置光标位置

## 3.1 可编辑div设置光标位置

```javascript
//设置光标位置
const setCaretPosition = function (element, pos) {
  var range, selection;
  if (document.createRange)//Firefox, Chrome, Opera, Safari, IE 9+
  {
    range = document.createRange();//创建一个选中区域
    range.selectNodeContents(element);//选中节点的内容
    if(element.innerHTML.length > 0) {
      range.setStart(element.childNodes[0], pos); //设置光标起始为指定位置
    }
    range.collapse(true);       //设置选中区域为一个点
    selection = window.getSelection();//获取当前选中区域
    selection.removeAllRanges();//移出所有的选中范围
    selection.addRange(range);//添加新建的范围
  }
  else if (document.selection)//IE 8 and lower
  {
    range = document.body.createTextRange();//Create a range (a range is a like the selection but invisible)
    range.moveToElementText(element);//Select the entire contents of the element with the range
    range.collapse(false);//collapse the range to the end point. false means collapse to end rather than the start
    range.select();//Select the range (make it the visible selection
  }
}
```

## 3.2 input/textarea获取光标位置

```javascript
// 设置光标位置
function setCaretPosition(textDom, pos){
    if(textDom.setSelectionRange) {
        // IE Support
        textDom.focus();
        textDom.setSelectionRange(pos, pos);
    }else if (textDom.createTextRange) {
        // Firefox support
        var range = textDom.createTextRange();
        range.collapse(true);
        range.moveEnd('character', pos);
        range.moveStart('character', pos);
        range.select();
    }
}
```

# 4.示例

```html
<html>
<head>
  <title>光标测试</title>
  <style>
    p {
      display: flex;
      flex-direction: row;
    }
    .btn {
      height: 24px;
      margin: 0 10px;
    }
    .edit-div {
      display: inline-block;
      width: 225px;
      border: 1px solid #decdcd;
    }
  </style>
  <script>
    function getCursortPosition(e) {
      var eleP = e.target.parentNode; //获取父级元素
      var pos = 0;
      if (e.target.nodeName == "DIV") {
        pos = getDivPosition(e.target);
      } else {
        pos = getPosition(e.target);
      }
      var spanEle = (eleP.childNodes)[7];
      spanEle.innerText = pos;
    }
    //可编辑div获取坐标
    const getDivPosition = function (element) {
      var caretOffset = 0;
      var doc = element.ownerDocument || element.document;
      var win = doc.defaultView || doc.parentWindow;
      var sel;
      if (typeof win.getSelection != "undefined") {//谷歌、火狐
        sel = win.getSelection();
        if (sel.rangeCount > 0) {//选中的区域
          var range = win.getSelection().getRangeAt(0);
          var preCaretRange = range.cloneRange();//克隆一个选中区域
          preCaretRange.selectNodeContents(element);//设置选中区域的节点内容为当前节点
          preCaretRange.setEnd(range.endContainer, range.endOffset);  //重置选中区域的结束位置
          caretOffset = preCaretRange.toString().length;
        }
      } else if ((sel = doc.selection) && sel.type != "Control") {//IE
        var textRange = sel.createRange();
        var preCaretTextRange = doc.body.createTextRange();
        preCaretTextRange.moveToElementText(element);
        preCaretTextRange.setEndPoint("EndToEnd", textRange);
        caretOffset = preCaretTextRange.text.length;
      }
      return caretOffset;
    }
    //输入框获取光标
    const getPosition = function (element) {
      let cursorPos = 0;
      if (document.selection) {//IE
        var selectRange = document.selection.createRange();
        selectRange.moveStart('character', -element.value.length);
        cursorPos = selectRange.text.length;
      } else if (element.selectionStart || element.selectionStart == '0') {
        cursorPos = element.selectionStart;
      }
      return cursorPos;
    }
  </script>
</head>
<body>
    <p>
    <label>输入框测试:</label>
    <input type="text" style="width:220px" onclick="getCursortPosition(event);" />
    <span>光标位置:</span>
    <span></span>
  </p>
  <p>
    <label>文本框测试:</label>
    <textarea rows="5" style="width:220px" onclick="getCursortPosition(event);"></textarea>
    <span>光标位置:</span>
    <span></span>
  </p>
  <div>
    <label>可编辑div:</label>
    <div contenteditable="true" class="edit-div" onclick="getCursortPosition(event);"></div>
    <span>光标位置:</span>
    <span></span>
  </div>
</body>
</html>
```