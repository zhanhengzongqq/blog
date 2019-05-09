- [问题表现](#%E9%97%AE%E9%A2%98%E8%A1%A8%E7%8E%B0)
- [问题排查](#%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5)
- [问题原因](#%E9%97%AE%E9%A2%98%E5%8E%9F%E5%9B%A0)
- [问题处理&&问题总结](#%E9%97%AE%E9%A2%98%E5%A4%84%E7%90%86%E9%97%AE%E9%A2%98%E6%80%BB%E7%BB%93)
- [相关截图](#%E7%9B%B8%E5%85%B3%E6%88%AA%E5%9B%BE)
- [相关代码](#%E7%9B%B8%E5%85%B3%E4%BB%A3%E7%A0%81)
  
  
### 问题表现
> 自动化发布之后系统502

### 问题排查
1. 发现 web.config 多了一个aspNetCore配置
2. 发现processPath 项分别指向了自己和同事的本机目录

### 问题原因
> Asp.net Core选择IIS启动的话，会在web.config下面生成一个system.webServer配置，具体配置与项目路径和IIS环境相关，如果大家都选择IIS启动则会产生多个配置，从而出错

### 问题处理&&问题总结
> 目前先使用项目启动，选择IIS启动只是作为本地hosts调试用，并不提交，还在寻找合适方案

### 相关截图

![Markdown](images/01.png?raw=true)


### 相关代码
> web.config配置（两个system.webServer）
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="C:\Program Files\dotnet\dotnet.exe" arguments="exec &quot;C:\Project\2019\src\master\CCMS.Host\bin\Debug\netcoreapp2.2\CCMS.Host.dll&quot;" stdoutLogEnabled="true" hostingModel="InProcess">
        <environmentVariables>
          <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
        </environmentVariables>
      </aspNetCore>
    </system.webServer>
  </location>
  <system.webServer>
    <handlers>
      <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModule" resourceType="Unspecified" />
    </handlers>
    <aspNetCore processPath="bin\IISSupport\VSIISExeLauncher.exe" arguments="-argFile IISExeLauncherArgs.txt" forwardWindowsAuthToken="false" stdoutLogEnabled="false" />
  </system.webServer>
</configuration>
```
