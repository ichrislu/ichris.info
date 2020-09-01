---
title: "go开发windows应用添加图标"
date: "2020-05-11 20:50:03"
lastMod: "2020-05-12 10:43:03"
tags: ["go", "manifest", "rsrc"]
---

开源工具源码地址：github.com/akavel/rsrc

- 下载rsrc工具
- 创建一个manifest文件

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
<assemblyIdentity version="1.0.0.0" processorArchitecture="Amd64" name="controls" type="win32"></assemblyIdentity>
<dependency>
    <dependentAssembly>
        <assemblyIdentity type="win32" name="Microsoft.Windows.Common-Controls" version="6.0.0.0" processorArchitecture="Amd64" publicKeyToken="6595b64144ccf1df" language="*"></assemblyIdentity>
    </dependentAssembly>
</dependency>
</assembly>
```

- windows下生成syso文件

```powershell
rsrc_windows_amd64.exe -manifest manifest -ico inventory.ico -arch amd64 -o inventory.syso
```

- 构建

```shell
GOOS=windows CGO_ENABLED=1 GOARCH=amd64 CC=x86_64-w64-mingw32-gcc CXX=x86_64-w64-mingw32-g++ go build -ldflags '-w -s' .
```



注意，网上找到的资料都会报以下的错：

```shell
/usr/local/go/pkg/tool/darwin_amd64/link: running x86_64-w64-mingw32-gcc failed: exit status 1
/usr/local/Cellar/mingw-w64/7.0.0_2/toolchain-x86_64/bin/x86_64-w64-mingw32-ld: i386 architecture of input file `/var/folders/lt/njzpfsqn63qbq509mzkkc9b80000gn/T/go-link-967461736/000000.o' is incompatible with i386:x86-64 output
collect2: 错误：ld 返回 1
```

多方找资料没有结果，最终查看rsrc官方说明，加上-arch参数解决问题，见生成syso文件命令部分