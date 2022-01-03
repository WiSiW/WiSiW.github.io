---
title: 【Echarts】读取本地Excel生成Echarts
date: 2021-01-27 15:54:50
tags:
    - Echarts
    - Excel
cover_picture: /images/echarts.png
---

![需要被读取的excel文件](data:image/png;base64,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)

[源码](https://github.com/WiSiW/frontend/tree/master/ExcelToCharts)

# 1.引用插件

```html
<script src="./xlsx.full.min.js"></script>
```
或
```
yarn add xlsx
import XLSX from 'xlsx'
```
# 2.上传本地文件插件

```javascript
var File = (function () {
    var that;
    var obj = function () {
        that = this;

        window.addEventListener("load",() => {
            that._init()
        }, false);
    }

    obj.prototype = {
        upload: function (options) {
            that._initByOptions(options);
            that.fileInput.click();
        },
        // 初始化上传框
        _init: function () {
            var ipt = document.createElement("input");
            ipt.style.display = "none";
            ipt.setAttribute("type", "file");
            ipt.addEventListener("change", that._fileChange, false);
            document.body.appendChild(ipt);

            that.fileInput = ipt;
        },

        _initByOptions: function (options) {
            that.fileInput.value = "";
            that.options = {
                multi: false,
                url: "",
                accept: "",
                param: null,
                uploadType:'',
                before: function () {
                },
                after: function () {
                },
                progress: function () {
                }
            };

            if (options) {
                for (var i in options) {
                    that.options[i] = options[i];
                }
            }

            // multiple
            if (that.options.multi) that.fileInput.setAttribute("multiple", "multiple");
            else that.fileInput.removeAttribute("multiple");

            // accept
            that.fileInput.setAttribute("accept", that.options.accept || "");
        },

        _fileChange: function () {
            if (that.options.before) {
                var result = that.options.before(this.files);
                if (result == false) return;
            }

            for (var i = 0; i < this.files.length; i++) {
                that._uploadFile(this.files[i]);
            }
        },

        _uploadFile: function (file) {
            if(that.options.uploadType == 'local'){
                that.options.after && that.options.after(file);
                return;
            }
            var xhr = new XMLHttpRequest();

            xhr.upload.addEventListener("progress", function (evt) {
                if (evt.lengthComputable) {
                    that.options.progress && that.options.progress(evt.loaded / evt.total);
                } else {
                    // No data to calculate on
                }
            }, false);

            xhr.addEventListener("load", function () {

            }, false);

            xhr.open("post", that.options.url);

            xhr.onreadystatechange = function () {
                if (xhr.readyState === 4) {
                    if (xhr.status >= 200 && xhr.status < 300 || xhr.status === 304) {
                        that.options.after && that.options.after(xhr.responseText, file);
                        xhr = null;
                    }
                }
            }

            xhr.onerror = function () {
                that.options.error && that.options.error();
            }

            // Send the file (doh)
            if ("getAsBinary" in file) {
                //FF 3.5+
                xhr.sendAsBinary(file.getAsBinary());
            } else {
                var formData = new FormData();
                formData.append("path", "default");
                formData.append("upload_file", file);
                formData.append("type", file.name.substring(file.name.indexOf(".")));

                if (that.options.param) {
                    for (var p in that.options.param) {
                        formData.append(p, that.options.param[p]);
                    }
                }

                if (me.global.token) {
                    xhr.setRequestHeader('token', me.global.token)
                }
                xhr.send(formData);
            }
        }
    }

    return new obj();
})();
```

# 3.读取Excel

```javascript
function XLSXRender(data, accept, maxSize, callBack, maxCount) {
  File.upload({
    multi: true,
    accept: accept,
    uploadType: "local",
    before: function (files) {},
    after: function (file) {
      var reader = new FileReader();
      reader.onload = function (e) {
        var data = e.target.result;
        var workbook = XLSX.read(data, {type: 'binary'});
        var sheetNames = workbook.SheetNames; // 工作表名称集合
        var worksheet = workbook.Sheets[sheetNames[0]]; // 这里我们只读取第一张sheet
        callBack && callBack(XLSX.utils.sheet_to_json(worksheet));
      };
      reader.readAsBinaryString(file);
    },
    error: function () {},
    progress: function () {}
  });
}
```

# 4.生成echarts
```javascript
XLSXRender({}, "image/jpg;image/jpeg;image/png;image/gif;image/bmp;", 500, function (data) {
  if (!data) return;
  var source = [];
  var a = {};
  for(let i in data){
    for(let key in data[i]) {
      if(!a[key])a[key] = [key]
      a[key].push(data[i][key])
    }
  }
  for(let i in a){
    source.push(a[i])
  }
  initData(source)
})
function initData(source){
  const _data = document.getElementById("data");
  const option = {
    legend: {},
    tooltip: {},
    dataset: {
      source: source
    },
    xAxis: {type: 'category'},
    yAxis: {},
    series: [
      {type: 'bar'},
      {type: 'bar'},
      {type: 'bar'}
    ]
  }
  if (!_data) {
    _data.setOption(option)
    return;
  }
  var _data = this.$echarts.init(_data)
  _data.setOption(option)
}
```

# 完整代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>ExcelToCharts</title>
    <script src="./xlsx.full.min.js"></script>
    <script src="./echarts.common.min.js"></script>
    <script src="./File.js"></script>
</head>
<body>
<button id="readExcel">读取Excel</button>
<div id="data" style="width: 300px;height: 300px;"></div>
<script>
    document.getElementById('readExcel').addEventListener('click', function() {
        XLSXRender({}, "*.xlsx;", 500, function (data) {
            if (!data) return;
            const source = [];
            const a = {};
            for(let i in data){
                for(let key in data[i]) {
                    if(!a[key])a[key] = [key]
                    a[key].push(data[i][key])
                }
            }
            for(let i in a){
                source.push(a[i])
            }
            initData(source)
        })
    })

    function XLSXRender(data, accept, maxSize, callBack, maxCount) {
        File.upload({
            multi: true,
            accept: accept,
            uploadType: "local",
            before: function (files) {},
            after: function (file) {
                var reader = new FileReader();
                reader.onload = function (e) {
                    var data = e.target.result;
                    var workbook = XLSX.read(data, {type: 'binary'});
                    var sheetNames = workbook.SheetNames; // 工作表名称集合
                    var worksheet = workbook.Sheets[sheetNames[0]]; // 这里我们只读取第一张sheet
                    callBack && callBack(XLSX.utils.sheet_to_json(worksheet));
                };
                reader.readAsBinaryString(file);
            },
            error: function () {},
            progress: function () {}
        });
    }

    function initData(source){
        const option = {
            legend: {},
            tooltip: {},
            dataset: {
                source: source
            },
            xAxis: {type: 'category'},
            yAxis: {},
            series: [
                {type: 'bar'},
                {type: 'bar'},
                {type: 'bar'}
            ]
        }

        let _data = document.getElementById("data");

        if (!_data) {
            _data.setOption(option)
            return;
        }
        _data = echarts.init(_data)

        _data.setOption(option)
    }
</script>
</body>
</html>
```

