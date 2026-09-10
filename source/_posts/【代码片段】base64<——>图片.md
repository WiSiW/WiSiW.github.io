---

title: 【代码片段】base64<——>图片
date: 2021-11-04 17:45:00
tags:
    - NodeJs
cover_picture: /images/node.png
---

# 1.base64转图片

```javascript
var app = require('express')();
app.post('/upload', function(req, res){
    //接收前台POST过来的base64
    var imgData = req.body.imgData;
    //过滤data:URL
    var base64Data = imgData.replace(/^data:image\/\w+;base64,/, "");
    var dataBuffer = new Buffer(base64Data, 'base64'); // 解码图片
    // var dataBuffer = Buffer.from(base64Data, 'base64'); // 这是另一种写法
    fs.writeFile("image.png", dataBuffer, function(err) {
        if(err){
          res.send(err);
        }else{
          res.send("保存成功！");
        }
    });
});
```

# 2.url转base64

```javascript
var app = require('express')();
app.get(url,function(res){
　　var chunks = []; //用于保存网络请求不断加载传输的缓冲数据
　　var size = 0;　　 //保存缓冲数据的总长度

　　res.on('data',function(chunk){
　　　　chunks.push(chunk);　 //在进行网络请求时，会不断接收到数据(数据不是一次性获取到的)
　　　　size += chunk.length;　　//累加缓冲数据的长度
　　});

　　res.on('end',function(err){
　　　　var data = Buffer.concat(chunks, size);　　//Buffer.concat将chunks数组中的缓冲数据拼接起来，返回一个新的Buffer对象赋值给data
　　　　var base64Img = data.toString('base64');　　//将Buffer对象转换为字符串并以base64编码格式显示
　　　　console.log(base64Img);　　 //进入终端terminal,然后进入index.js所在的目录，
　　});
});
```

# 3.图片转base64

```javascript
const fs = require('fs');
let bitmap = fs.readFileSync('start.jpg');

let base64str = Buffer.from(bitmap, 'binary').toString('base64'); // base64编码
console.log(base64str);
```
