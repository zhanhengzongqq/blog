

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