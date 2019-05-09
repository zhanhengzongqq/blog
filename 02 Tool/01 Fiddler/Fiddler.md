- [Fiddler捕获 Asp.net 程序中发送的请求的几种实现](#fiddler%E6%8D%95%E8%8E%B7-aspnet-%E7%A8%8B%E5%BA%8F%E4%B8%AD%E5%8F%91%E9%80%81%E7%9A%84%E8%AF%B7%E6%B1%82%E7%9A%84%E5%87%A0%E7%A7%8D%E5%AE%9E%E7%8E%B0)
  - [方法1 web.config 添加以下配置](#%E6%96%B9%E6%B3%951-webconfig-%E6%B7%BB%E5%8A%A0%E4%BB%A5%E4%B8%8B%E9%85%8D%E7%BD%AE)
  - [方法2 使用WebProxy代理](#%E6%96%B9%E6%B3%952-%E4%BD%BF%E7%94%A8webproxy%E4%BB%A3%E7%90%86)
- [Fiddler 的一些资料](#fiddler-%E7%9A%84%E4%B8%80%E4%BA%9B%E8%B5%84%E6%96%99)
  


###  Fiddler捕获 Asp.net 程序中发送的请求的几种实现

#### 方法1 web.config 添加以下配置
[参考于StackOverFlow](https://stackoverflow.com/questions/4629800/how-to-use-fiddler-to-monitor-wcf-service)

``` xml
   <system.net>
     <defaultProxy>
       <proxy bypassonlocal="False" usesystemdefault="True" proxyaddress="http://localhost:8888" />
     </defaultProxy>
   </system.net>
```

#### 方法2 使用WebProxy代理
[参考于StackOverFlow](https://stackoverflow.com/questions/16526689/using-a-proxy-with-net-4-5-httpclient)

``` csharp
private HttpClient CreateHttpClient(CommandContext ctx, string sid) {
    var cookies = new CookieContainer();

    var handler = new HttpClientHandler {
        CookieContainer = cookies,
        UseCookies = true,
        UseDefaultCredentials = false,
        Proxy = new WebProxy("http://localhost:8888", false, new string[]{}),
        UseProxy = true,
    };

    // snip out some irrelevant setting of authentication cookies

    var client = new HttpClient(handler) {
        BaseAddress = _prefServerBaseUrl,
    };

    client.DefaultRequestHeaders.Accept.Add(
        new MediaTypeWithQualityHeaderValue("application/json"));

    return client;
}

```
### Fiddler 的一些资料

[Fiddler教材（小坦克)](http://www.cnblogs.com/TankXiao/archive/2012/02/06/2337728.html)

[官方插件](https://www.telerik.com/fiddler/add-ons)

其中 Request to Code 比较好用
![](images/01.png?raw=true)