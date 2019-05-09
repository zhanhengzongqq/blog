
### 调用
``` javasript
    $.formPost('/DisabilityInfo/ExportExcel', searchData);
```
### 方法
``` javascript
//上传图片错误
$.extend({ 
    //包装后的form表单提交
    formPost: function (url, params) {
        var temp = document.createElement("form");
        temp.action = url;
        temp.method = "post";
        temp.style.display = "none";
        debugger
        for (var x in params) {
            if (params.hasOwnProperty(x)) {
                //TODO:Array.isArray 浏览器兼容性还需要测试
                if (Array.isArray(params[x]) && params[x].length > 1) {
                    //list处理
                    for (var i in params[x]) {
                        var opt = document.createElement("textarea");
                        opt.name = x + '[]';
                        opt.value = params[x][i];
                        temp.appendChild(opt);
                    }
                } else {
                    var opt = document.createElement("textarea");
                    opt.name = x;
                    opt.value = params[x];
                    temp.appendChild(opt);
                }
               
            }
        }
        document.body.appendChild(temp);
        temp.submit();
        return temp;
    }

});
```
### 如果不包含list处理
>Content-Type: application/x-www-form-urlencoded

![后台无法接受](/images/0010.png?raw=true)

![后台可以接受](/images/0011.png?raw=true)


