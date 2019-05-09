- [调用](#%E8%B0%83%E7%94%A8)
- [Jquery 扩展方法](#jquery-%E6%89%A9%E5%B1%95%E6%96%B9%E6%B3%95)
- [原来不包含list处理对于HTTP请求的影响](#%E5%8E%9F%E6%9D%A5%E4%B8%8D%E5%8C%85%E5%90%ABlist%E5%A4%84%E7%90%86%E5%AF%B9%E4%BA%8Ehttp%E8%AF%B7%E6%B1%82%E7%9A%84%E5%BD%B1%E5%93%8D)
- [改造后对于list处理对于HTTP请求的影响](#%E6%94%B9%E9%80%A0%E5%90%8E%E5%AF%B9%E4%BA%8Elist%E5%A4%84%E7%90%86%E5%AF%B9%E4%BA%8Ehttp%E8%AF%B7%E6%B1%82%E7%9A%84%E5%BD%B1%E5%93%8D)
  
  
### 调用
>  $.formPost('/DisabilityInfo/ExportExcel', searchData);

### Jquery 扩展方法
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
### 原来不包含list处理对于HTTP请求的影响
>Content-Type: application/x-www-form-urlencoded

![后台无法接受](/images/0010.png?raw=true)

### 改造后对于list处理对于HTTP请求的影响

![后台可以接受](/images/0011.png?raw=true)


