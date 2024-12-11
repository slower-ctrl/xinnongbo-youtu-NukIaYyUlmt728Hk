
一转眼好几年没有写博客了，来博客园冒个泡，最近由于工作需要，内网办公，幸运的是只需要上传一个\*.nupkg一个包信息就可以在私有nuget下载到了，下面就用PowerShell编写下载脚本，需要注意的是PowerShell后缀**ps1**（最后一个数字1），以Newtonsoft.Json为例：


## 下载地址




```
# 设置NuGet包列表的URL
$packageName = "Newtonsoft.Json"
$targetHttp = "https://www.nuget.org/packages/"
$targetUrl = "{0}{1}" -f $targetHttp, $packageName
```


## 保存地址




```
# 设置保存已下载包的目录
$outputDirectory = "D:\nuget_packages"
if (-not (Test-Path $outputDirectory)) {
    New-Item -Path $outputDirectory -ItemType Directory
}
```


## 解析下载版本地址


定义下载需要解析的包地址




```
# 定义下载前缀
$httpPrefix = "https://www.nuget.org/api/v2/package/"
# 下载html文件内容
$htmlContent = Invoke-WebRequest -Uri $targetUrl -UseBasicParsing | Select-Object -ExpandProperty Content
# 匹配标签
$pattern = "<.*?>"
$matches = [regex]::Matches($htmlContent, $pattern)
```


获取所有a标签




```
foreach ($match in $matches) {
　　$tag = $match.Value
　　# 获取a标签
　　if ($tag -like "[") {
　　　　Write-Host $tag
　　}
}](*</span)
```


 输出结果




```
<a href="#" id="skipToContent" class="showOnFocus" title="Skip To Content">
...
<a href="/packages/System.Xml.XmlDocument/">
<a href="/packages/Newtonsoft.Json/13.0.3" title="13.0.3">
...
<a href="/packages/Newtonsoft.Json/3.5.8" title="3.5.8">
<a href="/stats/packages/Newtonsoft.Json?groupby=Version" title="Package Statistics">
...
<a href="/packages/Newtonsoft.Json/13.0.3/ReportAbuse" title="Report the package as abusive">
<a href="/packages/Newtonsoft.Json/13.0.3/ContactOwners" title="Ask the package owners a question">
...
```


观察上一步结果可以看出来每一个版本都有title，且title内容是版本




```
# 获取含有title的a标签
if ($tag -like "*title=*") {
  Write-Host $tag        
}
```


 输出结果




```
<a href="#" id="skipToContent" class="showOnFocus" title="Skip To Content">
<a href="/packages/Newtonsoft.Json/13.0.3" title="13.0.3">
...
<a href="/packages/Newtonsoft.Json/3.5.8" title="3.5.8">
<a href="/stats/packages/Newtonsoft.Json?groupby=Version" title="Package Statistics">
<a href="https://www.newtonsoft.com/json" data-track="outbound-project-url" title="Visit the project site to learn more about this package" rel="nofollow">...
<a href="/packages/Newtonsoft.Json/13.0.3/ReportAbuse" title="Report the package as abusive">
<a href="/packages/Newtonsoft.Json/13.0.3/ContactOwners" title="Ask the package owners a question">
<a href="/packages?q=Tags%3A%22json%22" title="Search for json" class="tag">
```


接着上一步的结果继续过滤




```
# 截取href的内容
$substr = $tag.Substring(9)
if ($substr -like "/packages/*") {
    Write-Host $substr
}
```


输出结果




```
/packages/Newtonsoft.Json/13.0.3" title="13.0.3">
...
/packages/Newtonsoft.Json/3.5.8" title="3.5.8">
/packages/Newtonsoft.Json/13.0.3/ReportAbuse" title="Report the package as abusive">
/packages/Newtonsoft.Json/13.0.3/ContactOwners" title="Ask the package owners a question">
```


有完没完，又来了？看上面的结果就还差过滤两个不相关的了，


![](https://img2024.cnblogs.com/blog/1330501/202412/1330501-20241210214839965-1019945042.gif)


 获取href完整内容




```
# 查找第一个双引号的位置
$index = $substr.IndexOf('"')
# 获取部分/packages/Newtonsoft.Json/13.0.3
$substr = $substr.Substring(0,$index)
```


剔除最后两个版本无关的a标签




```
# 排除报告滥用a标签if ($substr -notlike "*/ReportAbuse") {
　　# 排除联系作者a标签
　　if ($substr -notlike "*/ContactOwners") {
　　　　# 匹配版本　　　　$endIndex = $substr.LastIndexOf('/')
　　　　$startPackageIndex = $endIndex + 1
　　　　$packageVersion = $substr.Substring($startPackageIndex)
　　}
}
```


## Invoke\-WebRequest命令下载并保存文件




```
# 下载地址nupkg$packageUrl = "{0}{1}/{2}" -f $httpPrefix,$packageName,$packageVersion
# 生成保存文件的路径
$packageFile = Join-Path -Path $outputDirectory -ChildPath "$packageName.$packageVersion.nupkg"
# 下载 .nupkg 文件
Write-Host "Downloading $packageName version $packageVersion from $packageUrl"
Invoke-WebRequest -Uri $packageUrl -OutFile $packageFile
```


全部代码


![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)


```
# 设置NuGet包列表的URL
$packageName = "Newtonsoft.Json"
$targetHttp = "https://www.nuget.org/packages/"
$targetUrl = "{0}{1}" -f $targetHttp, $packageName
# 设置保存已下载包的目录
$outputDirectory = "D:\nuget_packages"
if (-not (Test-Path $outputDirectory)) {
    New-Item -Path $outputDirectory -ItemType Directory
}
# 定义下载前缀
$httpPrefix = "https://www.nuget.org/api/v2/package/"
# 下载html文件内容
$htmlContent = Invoke-WebRequest -Uri $targetUrl -UseBasicParsing | Select-Object -ExpandProperty Content
# 匹配标签
$pattern = "<.*?>"
$matches = [regex]::Matches($htmlContent, $pattern)
foreach ($match in $matches) {
    $tag = $match.Value
        # 获取a标签
        if ($tag -like "[") {
            # 获取含有title的a标签
            if ($tag -like "*title=*")
            {
                # 截取href的内容
                $substr = $tag.Substring(9)
                if ($substr -like "/packages/*")
                {
                    # 查找第一个双引号的位置
                    $index = $substr.IndexOf('"')
                    # 获取部分/packages/Newtonsoft.Json/13.0.3
                    $substr = $substr.Substring(0,$index)
                    # 排除报告滥用a标签
                    if ($substr -notlike "*/ReportAbuse")
                    {
                        # 排除联系作者a标签
                        if ($substr -notlike "*/ContactOwners")
                        {
                            # 匹配版本
                            $endIndex = $substr.LastIndexOf('/')
                            $startPackageIndex = $endIndex + 1
                            $packageVersion = $substr.Substring($startPackageIndex)
                            # 下载地址nupkg
                            $packageUrl = "{0}{1}/{2}" -f $httpPrefix,$packageName,$packageVersion
                            # 生成保存文件的路径
                            $packageFile = Join-Path -Path $outputDirectory -ChildPath "$packageName.$packageVersion.nupkg"
                            # 下载 .nupkg 文件
                            Write-Host "Downloading $packageName version $packageVersion from $packageUrl"
              Invoke-WebRequest -Uri $packageUrl -OutFile $packageFile
                            
                        }
                        
                    }
                    
                }
                
            }
            
        }
}
# 执行结束暂停
$null = Read-Host](*</span)
```


View Code
 


 本博客参考[wgetcloud加速器官网下载](https://longdu.org)。转载请注明出处！
