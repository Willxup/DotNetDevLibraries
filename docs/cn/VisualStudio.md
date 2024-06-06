# IntelliSense本地化

## NET6之前的版本汉化

1. 之前的版本，微软提供了相应的汉化包，可以在[本地化 IntelliSense 文件](https://dotnet.microsoft.com/zh-cn/download/intellisense) 中下载相应的汉化包。
2. 下载完成后，将得到一个压缩包，解压后，可以看到有三个文件夹：`Microsoft.NETCore.App.Ref`,`Microsoft.WindowsDesktop.App.Ref`以及`NETStandard.Library.Ref`，这三个文件夹对应的是不同框架的汉化包。
   ![dotnet-intellisense-3.1-zh-hans](https://images.willxup.top/i/2022/10/26/ntzbba.png)
3. 找到`dotnet`目录(一般dotnet目录为：`C:\Program Files (x86)\dotnet\`)下的`pack`文件夹，在该文件夹下，对应的三种框架的文件夹。
   ![dotnet-packs](https://images.willxup.top/i/2022/10/26/nwjekc.png)
4. 选择`Microsoft.NETCore.App.Ref`文件夹，打开该文件夹后，可以看到有不同版本的`.NET`，找到相应的版本后，将解压出的对应版本文件夹下的`zh-hans`文件夹复制进来即可。

## .NET6汉化

`.NET6`版本之后，微软不再提供汉化包，但是还有其他方法可以进行汉化。

1. 安装`islocalizer`工具
```shell
#启动CMD，用dotnet工具进行安装
dotnet tool install -g islocalizer
```
2. 安装`.NET6`汉化包
```shell
islocalizer install auto -m net6.0 -l zh-cn
```
该工具将会从`github`下载相应的汉化包进行安装，最好有`github`加速，加快安装速度。

- 命令参数说明：
	- `-m` 指定`.NET`版本
	- `-l` 指定区域(语言)
	- `cc` 指定内容对照类型
		- `OriginFirst` 原始内容在前
		- `LocaleFirst` 本地化内容在前
		- `None` 没有对照
	- `sl` 分隔符

- 可通过`islocalizer install -h`及`islocalizer build -h`查看更多命令

- 相关举例
1. 安装汉化包，原始内容在前，本地化内容在后
```shell
islocalizer install auto -m net6.0 -l zh-cn -cc OriginFirst
```
2. 自定义生成格式，以`----------`分隔原始内容和本地化内容
```shell
islocalizer build -m net6.0 -l zh-cn -cc OriginFirst -sl '----------'
```