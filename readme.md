# NuGet属性表导入助手



## 1. 关于NuGet属性表导入助手
我们知道，C++项目大致框架如下：

```xml
<Project DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
    <!--...-->
    <Import Project="$(VCTargetsPath)\Microsoft.Cpp.Default.props" />
    <!--...-->
    <!--Microsoft.Cpp.props之前-->
    <Import Project="$(VCTargetsPath)\Microsoft.Cpp.props" />
    <!--Microsoft.Cpp.props之后，比如说 ExtensionSettings ImportGroup，Shared ImportGroup 以及 PropertySheets ImportGroup-->
    <!--...-->
    <!--Microsoft.Cpp.targets之前，-->
    <Import Project="$(VCTargetsPath)\Microsoft.Cpp.targets" />
    <!--Microsoft.Cpp.targets之后，比如说 ExtensionTargets ImportGroup-->
    <!--...-->
</Project>
```

而 NuGet原生仅仅支持二个属性表导入时机（即项目开头时，与项目结束时），这对于C++工程来说实在是太少了。比如说我们希望将属性表插入PropertySheets ImportGroup就无法做到。
而新的 PackageReference 想操作 vcxproj 更是困难，而本项目就是解决这个问题。NuGet属性表导入助手提供了4组导入时机，让我们能充分利用C++特性！
* 在 $(VCTargetsPath)\Microsoft.Cpp.props 之前
* 在 $(VCTargetsPath)\Microsoft.Cpp.props 之后
* 在 $(VCTargetsPath)\Microsoft.Cpp.targets 之前
* 在 $(VCTargetsPath)\Microsoft.Cpp.targets 之后

### 1.1. 兼容性
兼容 Visual Studio 2010及其以上所有IDE（不关心平台工具集版本）。

|       导入时机             |   最低支持IDE      |    最佳支持IDE
|      :--------:            |  :-----------:     |   :-----------:
| Microsoft.Cpp.props 之前   | Visual Studio 2019 | Visual Studio 2019
| Microsoft.Cpp.props 之后   | Visual Studio 2010 | Visual Studio 2019
| Microsoft.Cpp.targets 之前 | Visual Studio 2010 | Visual Studio 2010
| Microsoft.Cpp.targets 之后 | Visual Studio 2010 | Visual Studio 2010

### 1.2. 原理

#### 1.2.1. $(VCTargetsPath)\Microsoft.Cpp.props 之前导入
> 警告：Visual Studio 2019 IDE以及更高版本才能无缝使用此功能！

针对 Visual Studio 2019 IDE以及更高版本直接使用 `$(VCTargetsPath)\Microsoft.Cpp.props` 提供的 ForceImportBeforeCppProps 属性实现。

其他版本需要使用备选方案，在紧挨着 Microsoft.Cpp.props 之前手工添加，比如说：
```xml
<Import Condition="Exists('$(NuGetForceImportBeforeCppPropsFallback)')" Project="$(NuGetForceImportBeforeCppPropsFallback)" />
<!--备选方案，在紧挨着 Microsoft.Cpp.props 之前添加上面这一行-->
<Import Project="$(VCTargetsPath)\Microsoft.Cpp.props" />
```

#### 1.2.2. $(VCTargetsPath)\Microsoft.Cpp.props 之后导入
 
针对 Visual Studio 2019 IDE以及更高版本直接采用 `$(VCTargetsPath)\Microsoft.Cpp.props` 提供的 ForceImportAfterCppProps 属性实现。

其他版本通过 `Microsoft.Cpp.$(Platform).user.props` 跳转实现，如果您的工程不加载 `Microsoft.Cpp.$(Platform).user.props`，那么需要使用备选方案，在紧挨着 Microsoft.Cpp.props 之后手工添加，比如说：
```xml
<Import Project="$(VCTargetsPath)\Microsoft.Cpp.props" />
<!--备选方案，在紧挨着 Microsoft.Cpp.props 之后添加下面这一行-->
<Import Condition="Exists('$(NuGetForceImportAfterCppPropsFallback)')" Project="$(NuGetForceImportAfterCppPropsFallback)" />

```

#### 1.2.3. $(VCTargetsPath)\Microsoft.Cpp.targets 之前导入
直接使用 `$(VCTargetsPath)\Microsoft.Cpp.targets` 提供的 ForceImportBeforeCppTargets 属性实现。

#### 1.2.4. $(VCTargetsPath)\Microsoft.Cpp.targets 之后导入
直接使用 `$(VCTargetsPath)\Microsoft.Cpp.targets` 提供的 ForceImportAfterCppTargets 属性实现。


## 2. 使用方式
首先需要在 nuspec 清单中设置 dependency 设置依赖于 YY.NuGet.Import.Helper 包，比如说：

```xml
<!--您的 nuspec 文件-->
<package>
    <metadata>
        <!--...-->
        <dependencies>
            <dependency id="YY.NuGet.Import.Helper" version="1.0.0.4" />
            <!--...-->
        </dependencies>
    </metadata>
    <!--...-->
</package>
```

设置完毕后，您可以按下文所述方式使用。

### 2.1. 如何在 $(VCTargetsPath)\Microsoft.Cpp.props 之前导入?
比如说，您需要在这个时机导入 `C:\111.props`，那么请在的NuGet包里的props文件添加如下内容：
```xml
<!--{id}.props-->
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <NuGetImportBeforeCppProps Condition="Exists('C:\111.props')">$(NuGetImportBeforeCppProps);C:\111.props</NuGetImportBeforeCppProps>
  </PropertyGroup>
</Project>
```
> 请严格按上述内容使用，不要破坏 NuGetImportBeforeCppProps 链，以免影响到其他NuGet包。

### 2.2. 如何在 $(VCTargetsPath)\Microsoft.Cpp.props 之后导入？
比如说，您需要在这个时机导入 `C:\222.props`，那么请在的NuGet包里的props文件添加如下内容：
```xml
<!--{id}.props-->
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <NuGetImportAfterCppProps Condition="Exists('C:\222.props')">$(NuGetImportAfterCppProps);C:\222.props</NuGetImportAfterCppProps>
  </PropertyGroup>
</Project>
```
> 请严格按上述内容使用，不要破坏 NuGetImportAfterCppProps 链，以免影响到其他NuGet包。

### 2.3. 如何在 $(VCTargetsPath)\Microsoft.Cpp.targets 之前导入？
比如说，您需要在这个时机导入 `C:\333.props`，那么请在的NuGet包里的props文件添加如下内容：
```xml
<!--{id}.props-->
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <NuGetImportBeforeCppTargets Condition="Exists('C:\333.props')">$(NuGetImportBeforeCppTargets);C:\333.props</NuGetImportBeforeCppTargets>
  </PropertyGroup>
</Project>
```
> 请严格按上述内容使用，不要破坏 NuGetImportBeforeCppTargets 链，以免影响到其他NuGet包。

### 2.4. 如何在 $(VCTargetsPath)\Microsoft.Cpp.targets 之后导入？
比如说，您需要在这个时机导入 `C:\444.props`，那么请在的NuGet包里的props文件添加如下内容：
```xml
<!--{id}.props-->
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <NuGetImportAfterCppTargets Condition="Exists('C:\444.props')">$(NuGetImportAfterCppTargets);C:\444.props</NuGetImportAfterCppTargets>
  </PropertyGroup>
</Project>
```

> 请严格按上述内容使用，不要破坏 NuGetImportAfterCppTargets 链，以免影响到其他NuGet包。

### 2.5. 为 C++ NuGetPackge 添加 contentFiles 支持。

以 Detours 为例，我们只需在Detours.Static.targets中添加如下内容：

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup Label="Globals">
    <IncludePath>$(MSBuildThisFileDirectory);$(IncludePath)</IncludePath>
  </PropertyGroup>
  <ItemGroup>
    <NuGetExConnentFiles Include="$(MSBuildThisFileDirectory)Detours\*.cpp" Exclude="$(MSBuildThisFileDirectory)Detours\uimports.cpp">
      <!--NuGetPackageId必须这样写，否则罢工！-->
      <NuGetPackageId>$(MSBuildThisFileName)</NuGetPackageId>
      <!--配置为ClCompile项，一般来说，C与CPP都应该这样！-->
      <BuildAction>ClCompile</BuildAction>
      <!--关闭预编译头，强烈建议关闭，因为库一般不能使用这种玩意。-->
      <PrecompiledHeader>NotUsing</PrecompiledHeader>
      <!--关闭SDL，自己根据库决定，一般也推荐关闭。-->
      <SDLCheck>false</SDLCheck>
      <!--关闭符合模式，具体根据库决定，一般推荐关闭。-->
      <ConformanceMode>false</ConformanceMode>
      <!--配置obj输出目录，以免文件重名，强烈建议这样保留。-->
      <ObjectFileName>$(IntDir)$(MSBuildThisFileName)\</ObjectFileName>
      <!--关闭视警告为错误-->
      <TreatWarningAsError>false</TreatWarningAsError>
      <!--保持默认编译行为 .c 编译为 c，其他cpp-->
      <CompileAs>Default</CompileAs>
      <!--禁用CRT安全警告-->
      <PreprocessorDefinitions>_CRT_SECURE_NO_WARNINGS;%(PreprocessorDefinitions)</PreprocessorDefinitions>
    </NuGetExConnentFiles>
  </ItemGroup>
</Project>
```

Detours.Static.nuspec 内容如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<package xmlns="http://schemas.microsoft.com/packaging/2013/01/nuspec.xsd">
  <metadata minClientVersion="2.5">
    <id>Detours.Static</id>
    <version>4.0.1.19060</version>
    <title>Detours Source Package</title>
    <summary>Detours is a software package for monitoring and instrumenting API calls on Windows.</summary>
    <authors>Microsoft</authors>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <license type="expression">MIT</license>
    <projectUrl>https://github.com/microsoft/Detours</projectUrl>
    <description>Detours is a software package for monitoring and instrumenting API calls on Windows. Detours has been used by many ISVs and is also used by product teams at Microsoft. Detours is now available under a standard open source license (MIT). This simplifies licensing for programmers using Detours and allows the community to support Detours using open source tools and processes.

Detours is compatible with the Windows NT family of operating systems: Windows NT, Windows XP, Windows Server 2003, Windows 7, Windows 8, and Windows 10. It cannot be used by Window Store apps because Detours requires APIs not available to those applications. This repo contains the source code for version 4.0.1 of Detours.

For technical documentation on Detours, see the Detours Wiki. For directions on how to build and run samples, see the samples README.txt file.</description>
    <copyright>Copyright (c) Microsoft Corporation. All rights reserved.</copyright>
    <tags>Microsoft Detours native nativepackage</tags>
    <repository type="git" url="https://github.com/microsoft/Detours.git" branch="master" commit="edc8b07ae7e7325d9b9d551b46122a82665161b8" />
    <contentFiles>
      <!--故意添加一个 Detours.Static.txt（文件名需要跟包名一致），我们将通过它来检测 contentFiles 特性开启以及关闭-->
      <files include="any\any\Detours.Static.txt" buildAction="Content" copyToOutput="false"/>
    </contentFiles>
    <dependencies>
      <!--让C++ 包支持 contentFiles 建议使用 1.0.0.4 版本-->
      <dependency id="YY.NuGet.Import.Helper" version="1.0.0.4" />
    </dependencies>
  </metadata>
  <files>
    <file src=".\src\**" target="build\native\Detours" exclude=".\src\Makefile"/>
    <file src=".\detours.Static.targets" target="build\native\Detours.Static.targets"/>
    <!--Detours.Static.txt文件名跟包名一致即可，0 字节的文件。-->
    <file src=".\Detours.Static.txt" target="contentFiles\any\any\Detours.Static"/>
    <file src=".\Detours.Static.txt" target="Content\NuGet\Detours.Static"/>
  </files>
</package>
```

> 添加 contentFiles 特性后，对于PR（PackageReference）模式，可以通过 `<ExcludeAssets>contentFiles</ExcludeAssets>` 来阻止包中的内容编译到项目中。    
对于 PC（packages.config）模式，则可以把 Detours.Static.txt 设置为从生成中排除。即`<ExcludedFromBuild>true</ExcludedFromBuild>` 来阻止包中的内容编译到项目中。


## 3. 更新日志
[更新日志](https://github.com/mingkuang-Chuyu/YY.NuGet.Import.Helper/wiki)
