

#### 方法1 web.config 添加以下配置
[参考](https://stackoverflow.com/questions/4629800/how-to-use-fiddler-to-monitor-wcf-service)

``` xml
   <system.net>
     <defaultProxy>
       <proxy bypassonlocal="False" usesystemdefault="True" proxyaddress="http://localhost:8888" />
     </defaultProxy>
   </system.net>
```

#### 方法2 使用WebProxy代理
[参考](https://stackoverflow.com/questions/16526689/using-a-proxy-with-net-4-5-httpclient)

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
其他文章

[Fiddler教材（小坦克)](http://www.cnblogs.com/TankXiao/archive/2012/02/06/2337728.html)

[官方插件](https://www.telerik.com/fiddler/add-ons)

其中 Request to Code 比较好用
![](images/01.png?raw=true)