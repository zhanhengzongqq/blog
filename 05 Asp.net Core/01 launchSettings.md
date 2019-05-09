
- [IIS 启动设置配置（iisSettings）](#iis-%E5%90%AF%E5%8A%A8%E8%AE%BE%E7%BD%AE%E9%85%8D%E7%BD%AEiissettings)
- [配置文件（profiles）](#%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6profiles)


[参考官网](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/environments?view=aspnetcore-2.2)
### IIS 启动设置配置（iisSettings）
``` json
  "iisSettings": {
    "windowsAuthentication": false,  //启用windows身份认证
    "anonymousAuthentication": true, //启用匿名身份认证
    "iis": {    //外部IIS配置
      "applicationUrl": "http://test.acme.com/", //HTTP启动地址，
      "sslPort": 0 //HTTPS 端口
    },
    "iisExpress": { //VS自带IIS配置
      "applicationUrl": "http://localhost:54755",
      "sslPort": 44370
    }
  }
```

### 配置文件（profiles）
> 可为项目新建多个配置文件，在启动的时候选择其中一个配置文件

commandName 可为以下任一项
- IISExpress
- IIS
- Project（启动 Kestrel 的项目）

```json
  "profiles": {
    "NetCoreDemo": {  //配置名称
      "commandName": "IIS", //启动方式（IIS|IISExpress|Project|Executable）
      "launchBrowser": true, //运行启动浏览器
      "environmentVariables": { //环境变量，可设置多个
        "a": "b",
        "ASPNETCORE_ENVIRONMENT": "Development"
      },
      "applicationUrl": "http://localhost:5003" //启动项为“项目”是的启动地址默认为 http://localhost:5000
    },
    "newProfile1": { //其他配置文件
         ...
    }
  }
```
