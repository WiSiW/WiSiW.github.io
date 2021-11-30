---
title: 【canvas】canvas绘制签名
date: 2021-11-24 11:32:49
tags:
    - canvas
    - 绘图
    - 签名
---

# pc端、手机端适配canvas绘图

## 原理

**监听触摸滑动或鼠标滑动，在canvas上绘制出相应的笔迹**

## 功能演示

**HTML页面，下图灰色区域为绘制区，停止绘制后会在下方方框内生成图片**

![canvas绘制](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAANgAAAEOCAMAAADc7ilZAAAAsVBMVEX////39/fAwMAAAAAMDAzy8vKHh4dcXFz9/PzT09Pw7/DOzs6AgIB4eHgcHBz29vZISEgyMjLMzMwWFhaysrIqKiomJibCwsKZmZnd3d2Dg4NpaWni4uKsrKwuLi7t7e3p6enHx8eTk5OMjIxWVlZMTEwiIiLX19ejo6OgoKBOTk47Ozu3t7ecnJx2dnZxcXFBQUHR0NGoqKhSUlLk5OR7e3tkZGS9vb2kpKQ2NjZzc3MLCvRAAAADZElEQVR42u3X227aQBSF4UVmMCaxHYgxxeF8DgECgRz7/g/WsaVQpF5Aq0rOStZ3sa3x3W9tXwxEROQ7KX0RCmOjMDYKY6MwNgpjozA2CmOjMDYKY6MwNgpjozA2CmOjMDYKY6MwNgpjozA2CmOjMDYKY6MwNgpjozA2CmOjMDYKY6MwNgpjozA2CmOjMDYKY6MwNgpjozA2CmOjMDYKY6MwNgpjozA2CmOjMDYKY6MwNgpjozA2CmPzfcJERERERERE5B/4HeReV/htME7xl/xWenzyXlGwkeliFIZhkrjhAaVrpx9vrnM4zas4HsYmWCwa+FBJfBTL35jxwvO8XuLGCJiZIxFOW/w0led11WwnezMFsDNHZijQXdpvt9uzmRtVF7apHoQRzjA0T0jjmg/PdgC82ZuD5wiFafSBchAESeJG6sLGwHqM0nYHjCKcYTwD3k1Urxtbr5fxFuNgXVzYpe0N0lsn6WWziu68XzEhyhsz7VS7OEPtBbh5SVqVuPVsynidrOsfGhsUpho3G5WDIfqhsQ0486UZ+TgtNTaKVsEUdz08ujBg6HnexHhOCcVxAR3sWrlFdjRBHzk/jMo4rW22C9sN7H1i75d5WGlYQsPgM2gmNadp4Vz9OHjEaU/GhIi6QRRMbfCSh61M5zOE+dshmsGV4+Vhz+bgAaddL7dZ2PEqtmdwYTXnAcXJv2/TxI6xeWjZ2U+yWcIZ/Pss7Gd0u7e37SzMj1+ysAenhQJNakBzPHBa9vfLCs6Wh+2jac9M83/swtjPsIoDs3NhJvePYdOd7TZHH6tYSpbb907xYSNz7cLWvrPIwqpXmVotf1TPCrPNZmhusrChCwtM2l+ampkPSihS0kYW5oomcR1AbI5YnGEfApV3uLBNbP1LMwL8u3trch6KkqYAvCFw6Y3nAMqXx3CG7A7QuQKefty8zYGGj1z/cXixWj1BRERERERERD6Viy/izzB8CQpjozA2CmOjMDYKY6MwNgpjozA2CmOjMDYKY6MwNgpjozA2CmOjMDYKY6MwNgpjozA2CmOjMDYKY6MwNgpjozA2CmOjMDYKY6MwNgpjozA2CmOjMDYKY6MwNgpjozA2CmOjMDYKY6MwNgpjozA2CmOjMDYKY6MwNt8p7IuAiIiIiPw3vwAuu4X1Tm+PogAAAABJRU5ErkJggg==)

```html
<html>
    <head>
        <style>
            .sign-box {
                width: 200px;
            }
            .sign-box .sign-img {
                width: 100%;
                height: 100px;
            }
            .sign-box .btn-box>p {
                width: 60px;
                height: 20px;
                line-height: 20px;
                text-align: center;
                display: inline-block;
            }
        </style>
    </head>
    <body>
        <div class="sign-box">
            <div class="sign-img" id="canvas">
            </div>
            <div class="btn-box">
                <p id="btn-clear">清空</p>
                <p id="btn-sub">确定</p>
            </div>
            <img class="sign-img" id="img">
        </div>
    </body>
</html>
```

![canvas绘制后](data:image/png;base64,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)

## 绘制程序



```javascript
let cxt = null;
// 签名笔记的基础样式
const lineStyle = {
  lineWidth: 2,
  strokeStyle: '#000000',
  fillStyle: '#f7f7f7',
  lineCap: 'round'
}
// 绘制区
const canvasEl = document.getElementById("canvas")
// 清空按钮
const clearBtn = document.getElementById("btn-clear")
// 提交按钮
const subBtn = document.getElementById("btn-sub")
lineCanvas({canvasEl, clearBtn, subBtn})
// 移动绘制
function lineCanvas({ canvasEl, clearBtn, subBtn }) {
  const canvas = document.createElement("canvas");
  canvas.width = canvasEl.clientWidth;
  canvas.height = canvasEl.clientHeight;
  canvasEl.appendChild(canvas);
  cxt = canvas.getContext("2d");
  console.log(cxt)
  Object.keys(lineStyle).forEach(key => {
    cxt[key] = lineStyle[key];
  })
  cxt.fillRect(0, 0, canvas.width, canvas.height)
  IsPC() ? addMoveEvents(canvas) : addTouchEvents(canvas)
  // 为【清除】按钮绑定点击事件，清空画布
  clearBtn && clearBtn.addEventListener("click", function() {
    console.log("清空画布...");
    cxt.clearRect(0, 0, canvas.width, canvas.height);
    document.getElementById("img").src = '';
  })
  // 为【确定】按钮绑定点击事件保存图片
  subBtn && subBtn.addEventListener("click", function(e) {
    console.log("保存图片...");
    if(!signImg) {
      alert("请签名")
      return
    }
    // do somthing
  })
}
// 判断当前是否是PC端
function IsPC() {
  var userAgentInfo = navigator.userAgent;
  var Agents = ["Android", "iPhone", "SymbianOS", "Windows Phone", "iPad", "iPod"];
  var flag = true;
  for (var v = 0; v < Agents.length; v++) {
    if (userAgentInfo.indexOf(Agents[v]) > 0) {
      flag = false;
      break;
    }
  }
  return flag;
}
```

##  pc端鼠标按下绘制

```javascript
// 鼠标绘制
function addMoveEvents(canvas) {
  // 鼠标按下事件
  const mousedown = function(e) {
    console.log("开始绘制...");
    cxt.beginPath();
    cxt.moveTo(e.pageX, e.pageY);
    // 绘制中
    canvas.addEventListener("mousemove", mousemove, false)
  }
  // 鼠标按下移动绘制事件
  const mousemove = function(e) {
    console.log("绘制中...");
    cxt.lineTo(e.pageX, e.pageY);
    cxt.stroke();
  }
  // 鼠标松开事件
  const mouseup = function(e) {
    console.log("结束绘制...");
    canvas.removeEventListener("mousemove", mousemove)
    cxt.closePath();
    signImg = canvas.toDataURL();
    console.log(signImg)
    document.getElementById("img").src = signImg
  }
  // 阻止页面滑动
  window.addEventListener('touchmove', function(e) {
    e.preventDefault()
  }, { passive: false });
  // 监听绘制开始
  canvas.addEventListener("mousedown", mousedown, false)
  // 结束绘制
  canvas.addEventListener("mouseup", mouseup, false)
  // 鼠标移除画板后取消绘制时间
  canvas.addEventListener("mouseout", function(e) {
    console.log("鼠标移除画板");
    canvas.removeEventListener("mousemove", mousemove)
  })
}
```



## mobile端触摸绘制

```javascript
// 触摸绘制
function addTouchEvents(canvas) {
  // 手指点下开始事件
  const touchstart = function(e) {
    console.log("开始绘制...");
    cxt.beginPath();
    cxt.moveTo(e.changedTouches[0].pageX, e.changedTouches[0].pageY);
  }
  // 触摸移动绘制事件
  const touchmove = function(e) {
    console.log("绘制中...");
    cxt.lineTo(e.changedTouches[0].pageX, e.changedTouches[0].pageY);
    cxt.stroke();
  }
  // 手指移开事件
  const touchend = function(e) {
    console.log("结束绘制...");
    cxt.closePath();
    signImg = canvas.toDataURL();
    console.log(signImg)
    document.getElementById("img").src = signImg
  }
  // 监听绘制开始
  canvas.addEventListener("touchstart", touchstart, false)
  // 绘制中
  canvas.addEventListener("touchmove", touchmove, false)
  // 结束绘制
  canvas.addEventListener("touchend", touchend, false)
}
```



## 完整代码

```html
<html>
    <head>
        <style>
            .sign-box {
                width: 200px;
            }
            .sign-box .sign-img {
                width: 100%;
                height: 100px;
            }
            .sign-box .btn-box>p {
                width: 60px;
                height: 20px;
                line-height: 20px;
                text-align: center;
                display: inline-block;
            }
        </style>
    </head>
    <body>
        <div class="sign-box">
            <div class="sign-img" id="canvas">
            </div>
            <div class="btn-box">
                <p id="btn-clear">清空</p>
                <p id="btn-sub">确定</p>
            </div>
            <img class="sign-img" id="img">
        </div>
        <script>
            let cxt = null;
            const lineStyle = {
                lineWidth: 2,
                strokeStyle: '#000000',
                fillStyle: '#f7f7f7',
                lineCap: 'round'
            }
            const canvasEl = document.getElementById("canvas")
            const clearBtn = document.getElementById("btn-clear")
            const subBtn = document.getElementById("btn-sub")
            window.addEventListener('touchmove', function(e) {
                e.preventDefault()
            }, { passive: false });
            lineCanvas({canvasEl, clearBtn, subBtn})
            // 移动绘制
            function lineCanvas({ canvasEl, clearBtn, subBtn }) {
                const canvas = document.createElement("canvas");
                canvas.width = canvasEl.clientWidth;
                canvas.height = canvasEl.clientHeight;
                canvasEl.appendChild(canvas);
                cxt = canvas.getContext("2d");
                console.log(cxt)
                Object.keys(lineStyle).forEach(key => {
                    cxt[key] = lineStyle[key];
                })
                cxt.fillRect(0, 0, canvas.width, canvas.height)
                IsPC() ? addMoveEvents(canvas) : addTouchEvents(canvas)
                // 清空画布
                clearBtn && clearBtn.addEventListener("click", function() {
                    console.log("清空画布...");
                    cxt.clearRect(0, 0, canvas.width, canvas.height);
                    document.getElementById("img").src = '';
                })
                // 保存图片
                subBtn && subBtn.addEventListener("click", function(e) {
                    console.log("保存图片...");
                    if(!signImg) {
                        alert("请签名")
                        return
                    }
                    // do somthing
                })
            }
            // 鼠标绘制
            function addMoveEvents(canvas) {
                // 鼠标按下事件
                const mousedown = function(e) {
                    console.log("开始绘制...");
                    cxt.beginPath();
                    cxt.moveTo(e.pageX, e.pageY);
                    // 绘制中
                    canvas.addEventListener("mousemove", mousemove, false)
                }
                // 鼠标按下移动绘制事件
                const mousemove = function(e) {
                    console.log("绘制中...");
                    cxt.lineTo(e.pageX, e.pageY);
                    cxt.stroke();
                }
                // 鼠标松开事件
                const mouseup = function(e) {
                    console.log("结束绘制...");
                    canvas.removeEventListener("mousemove", mousemove)
                    cxt.closePath();
                    signImg = canvas.toDataURL();
                    console.log(signImg)
                    document.getElementById("img").src = signImg
                }
                // 监听绘制开始
                canvas.addEventListener("mousedown", mousedown, false)
                // 结束绘制
                canvas.addEventListener("mouseup", mouseup, false)
                // 鼠标移除画板后取消绘制时间
                canvas.addEventListener("mouseout", function(e) {
                    console.log("鼠标移除画板");
                    canvas.removeEventListener("mousemove", mousemove)
                })
            }
            // 触摸绘制
            function addTouchEvents(canvas) {
                // 手指点下开始事件
                const touchstart = function(e) {
                    console.log("开始绘制...");
                    cxt.beginPath();
                    cxt.moveTo(e.changedTouches[0].pageX, e.changedTouches[0].pageY);
                }
                // 触摸移动绘制事件
                const touchmove = function(e) {
                    console.log("绘制中...");
                    cxt.lineTo(e.changedTouches[0].pageX, e.changedTouches[0].pageY);
                    cxt.stroke();
                }
                // 手指移开事件
                const touchend = function(e) {
                    console.log("结束绘制...");
                    cxt.closePath();
                    signImg = canvas.toDataURL();
                    console.log(signImg)
                    document.getElementById("img").src = signImg
                }
                // 监听绘制开始
                canvas.addEventListener("touchstart", touchstart, false)
                // 绘制中
                canvas.addEventListener("touchmove", touchmove, false)
                // 结束绘制
                canvas.addEventListener("touchend", touchend, false)
            }
            function IsPC() {
                var userAgentInfo = navigator.userAgent;
                var Agents = ["Android", "iPhone", "SymbianOS", "Windows Phone", "iPad", "iPod"];
                var flag = true;
                for (var v = 0; v < Agents.length; v++) {
                    if (userAgentInfo.indexOf(Agents[v]) > 0) {
                        flag = false;
                        break;
                    }
                }
                return flag;
            }
        </script>
    </body>
</html>
```

