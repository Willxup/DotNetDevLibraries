<p align="center">
  <a href="../en/VisualStudio.md"><strong>English</strong></a> ｜ <a href="../cn/VisualStudio.md">简体中文</a>
</p>

[← Contents](../../README.md)

# IntelliSense Localization

## Chinese Localization Before .NET 6

1. For earlier versions, Microsoft provides Chinese language packages. Download the package you need from [localized IntelliSense files](https://dotnet.microsoft.com/zh-cn/download/intellisense).
2. Extract the downloaded archive. It contains three folders: `Microsoft.NETCore.App.Ref`, `Microsoft.WindowsDesktop.App.Ref`, and `NETStandard.Library.Ref`. Each contains Chinese language files for a different framework.
   ![dotnet-intellisense-3.1-zh-hans](https://images.willxup.top/i/2022/10/26/ntzbba.png)
3. Find the `pack` folder in the `dotnet` directory (usually `C:\Program Files (x86)\dotnet\`). It contains folders for the three frameworks.
   ![dotnet-packs](https://images.willxup.top/i/2022/10/26/nwjekc.png)
4. Open `Microsoft.NETCore.App.Ref` and find the .NET version you need. Copy the `zh-hans` folder from the matching version in the extracted archive into that folder.

## Chinese Localization for .NET 6

Starting with .NET 6, Microsoft no longer provides these Chinese language packages, but there are other ways to install them.

1. Install the `islocalizer` tool.
```shell
# Open CMD and install with dotnet tool
dotnet tool install -g islocalizer
```
2. Install the Chinese language package for `.NET6`.
```shell
islocalizer install auto -m net6.0 -l zh-cn
```
The tool downloads the Chinese language package from GitHub. A faster connection to GitHub can speed up installation.

- Command options:
	- `-m`: Select the `.NET` version.
	- `-l`: Select the locale (language).
	- `cc`: Select how the original and localized content are shown.
		- `OriginFirst`: Show the original content first.
		- `LocaleFirst`: Show the localized content first.
		- `None`: Do not show a comparison.
	- `sl`: Set the separator.

- Use `islocalizer install -h` and `islocalizer build -h` to see more commands.

- Examples:
1. Install the Chinese language package with the original content first and localized content second.
```shell
islocalizer install auto -m net6.0 -l zh-cn -cc OriginFirst
```
2. Generate a custom format that separates the original and localized content with `----------`.
```shell
islocalizer build -m net6.0 -l zh-cn -cc OriginFirst -sl '----------'
```
