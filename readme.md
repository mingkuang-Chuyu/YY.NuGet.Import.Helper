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
            <dependency id="YY.NuGet.Import.Helper" version="1.0.0.1" />
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
    <NuGetImportBeforeCppProps>$(NuGetImportBeforeCppProps);C:\111.props</NuGetImportBeforeCppProps>
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
    <NuGetImportAfterCppProps>$(NuGetImportAfterCppProps);C:\222.props</NuGetImportAfterCppProps>
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
    <NuGetImportBeforeCppTargets>$(NuGetImportBeforeCppTargets);C:\333.props</NuGetImportBeforeCppTargets>
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
    <NuGetImportAfterCppTargets>$(NuGetImportAfterCppTargets);C:\444.props</NuGetImportAfterCppTargets>
  </PropertyGroup>
</Project>
```

> 请严格按上述内容使用，不要破坏 NuGetImportAfterCppTargets 链，以免影响到其他NuGet包。

## 3. 更新日志
### v1.0.0.1 - 2020-01-02 首次发布
* 添加 $(VCTargetsPath)\Microsoft.Cpp.props 之前导入时机
* 添加 $(VCTargetsPath)\Microsoft.Cpp.props 之后导入时机
* 添加 $(VCTargetsPath)\Microsoft.Cpp.targets 之前导入时机
* 添加 $(VCTargetsPath)\Microsoft.Cpp.targets 之后导入时机