---
layout: post
title: 使用Nuget发布自己的package
author: kinshines
date:   2016-01-12
categories: c#
permalink: /archivers/nuget-create-package
---

<p class="lead">Nuget是Visual Studio的强大扩展，它为开发者提供了极为方便的开发体验，可以直接安装或更新第三方组件，甚至可以安装一些Visual Studio的插件。作为一名开发人员，我们可能也会开发一些公共组件以供他人使用，本文将一步步介绍如何以最简单的方式将自己所开发的类库包发布到nuget上，以供更多的人使用</p>

### 1.在NuGet上注册并获取API Key
到[NuGet](https://www.nuget.org/)上注册一个新的账号，然后在[My Account](https://www.nuget.org/account)页面，获取一个API Key，这个过程很简单，就不作说明了。

### 2.下载NuGet.exe
下载NuGet命令行工具：NuGet.exe，下载地址：[NuGet Distribution](https://dist.nuget.org/index.html) 下载下来后要设置机器的PATH环境变量，将NuGet.exe的路径添加到PATH中。

### 3.设置API Key
API key是你发布Nuget资源时token，记得保密哦，设置命令：

        nuget.exe setApiKey [your API key]

### 4.开发自己的类库
新建了一个类库：SimpleNugetTest，并通过NuGet添加了对Castle.Core的引用，现在我们添加一些代码，来使用Castle.Core所提供的一些功能。我们将Class1.cs改名为CastleHelper.cs，此时也会将Class1类改名为CastleHelper。在CastleHelper.cs中写入以下代码：
{% highlight java %}
public class CastleHelper
{
    public static Castle.Core.Pair<int, int> GetIntPair()
    {
        return new Castle.Core.Pair<int, int>(20, 30);
    }
}
{% endhighlight %}

然后，打开AssemblyInfo.cs文件，将assembly的属性设置好，记得再设置一下AssemblyVersion特性，以指定我们类库的版本。目前我们使用1.0.0.0版本：
{% highlight java %}
[assembly: AssemblyTitle("SimpleNugetTest")]
[assembly: AssemblyDescription("Simple test of the NuGet.")]
[assembly: AssemblyConfiguration("")]
[assembly: AssemblyCompany("")]
[assembly: AssemblyProduct("SimpleNugetTest")]
[assembly: AssemblyCopyright("Copyright © kinshines 2016")]
[assembly: AssemblyTrademark("")]
[assembly: AssemblyCulture("")]
 
[assembly: ComVisible(false)]
 
[assembly: Guid("20662b9f-91de-4515-9c8c-ced3d61589e1")]
 
[assembly: AssemblyVersion("1.0.0.0")]
{% endhighlight %}
全部设置好以后，编译整个项目待用。

### 5.产生并修改nuspec
nuspec是NuGet将项目打包成nupkg的输入文件，可以通过nuget spec命令产生。在命令提示符下，进入SimpleNugetTest.csproj文件所在目录，然后执行：

        nuget spec

提示创建成功：Created 'SimpleNugetTest.nuspec' successfully.

用notepad打开SimpleNugetTest.nuspec文件，把需要替换的信息替换掉，不需要的tag全部删掉，注意里面的$xxx$宏，这些就是引用了AssemblyInfo.cs中的设置值，在编译产生package的时候，会使用AssemblyInfo.cs中的相应值进行替换。完成编辑后，nuspec文件如下：
{% highlight xml %}
<?xml version="1.0"?>
<package >
  <metadata>
    <id>$id$</id>
    <version>$version$</version>
    <title>$title$</title>
    <authors>$author$</authors>
    <owners>$author$</owners>
    <licenseUrl>http://www.apache.org/licenses/LICENSE-2.0.html</licenseUrl>
    <projectUrl>http://apworks.org</projectUrl>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>$description$</description>
    <releaseNotes>First release</releaseNotes>
    <copyright>Copyright 2013</copyright>
  </metadata>
</package>
{% endhighlight %}

注意两点：
1. $description$使用AssemblyDescriptionAttribute的值进行替换，在产生package之前，一定要记得先编译项目，否则会提示$description$找不到的错误
2. releaseNotes如果没有，就直接删掉这个节点，如果有，则填入自己的内容，不要使用默认内容，否则会在下一步产生警告信息

### 6.产生类库包（Library Package）
同样在SimpleNugetTest.csproj路径下，使用下面的命令产生NuGet类库包：

        nuget pack SimpleNugetTest.csproj

成功后，提示：Successfully created package

注意：由于项目通过NuGet引用了Castle.Core，因此，它将会作为一个依赖组件（dependency）打包到产生的nupkg文件中。

另外，NuGet会使用默认的项目配置所产生的程序集进行打包。如果项目默认是Debug，而需要用Release打包，则使用下面的命令：

        nuget pack SimpleNugetTest.csproj -Prop Configuration=Release

### 7.发布类库包
现在，通过以下命令发布类库包：

        nuget push SimpleNugetTest.1.0.0.0.nupkg

完成后，出现提示：Your package was pushed.

### 8.测试已发布的类库
新建一个控制台应用程序，在项目上点右键，选择Manage NuGet Packages，在搜索框中输入SimpleNugetTest，此时发布的Package已经可以显示了，单击Install按钮，NuGet会自动分析组件依赖关系，然后把所需要的所有程序集都下载下来并添加到项目引用中

测试代码：
{% highlight java %}
class Program
{
    static void Main(string[] args)
    {
        var pair = SimpleNugetTest.CastleHelper.GetIntPair();
        Console.WriteLine(pair.First);
        Console.WriteLine(pair.Second);
    }
}
{% endhighlight %}

### 9.更新类库包
随着类库开发进度不断向前，必然会有版本更新。更新类库包很简单，只需要在AssemblyInfo.cs中更新一下版本号，然后重新执行上面的STEP 6、7即可。注意在执行STEP 7的时候，nupkg的文件名应该使用新版本的文件名。

现在，我们重新打开SimpleNugetTest项目，将CastleHelper类中的20,30改为40,50，然后打开AssemblyInfo.cs，版本号升级为2.0.0.0，重新编译项目，并重新产生、发布nupkg

### 10.删除已发布的包
原则上，NuGet不允许用户删除已发布的包，而只能将其设置为不显示在Manage NuGet Packages的列表中。打开[www.nuget.org](https://www.nuget.org/)，用已注册的账户登录后，可以在[My Account](https://www.nuget.org/account) 页面选择[Manage My Packages](https://www.nuget.org/account/Packages) 链接进入管理页面：
点击SimpleNugetTest左边的小垃圾桶图标，即可进入Listing页面，页面中我们也能看到“Permanently deleting packages is not supported”的提示。要将Package从Package List中移除，只需要去掉List SimpleNugetTest 2.0.0.0 in search results选项前面的钩钩，然后单击Save按钮保存即可

### 11.按照约定结构的文件夹制作Nuget包
上面讲解了使用Visual Studio project来制作简单的Nuget包，但在实际操作时，有一些像readme之类的文件也希望包含到Nuget包中，package中大体可以包含以下三类：

* Content and source code that should be injected into the target project
* PowerShell scripts (packages used in NuGet 2.x can include installation scripts as well, which is not supported in NuGet 3.x and later.)
* Transformations to existing configuration and source code files in a project

因此使用以下约定来布局文件夹结构：

<table>
  <thead>
    <tr>
      <th>Folder</th>
      <th>Description</th>
      <th>Action upon package install</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>tools</td>
      <td>Powershell scripts and programs accessible from the Package Manager Console</td>
      <td>Contents are copied to the project folder, and the tools folder is added to the PATH environment variable.</td>
    </tr>
    <tr>
      <td>lib</td>
      <td>Assembly files (.dll), documentation (.xml) files, and symbol (.pdb) files</td>
      <td>Assemblies are added as references; .xml and .pdb copied into project folders.</td>
    </tr>
    <tr>
      <td>content</td>
      <td>Arbitrary files</td>
      <td>Contents are copied to the project root</td>
    </tr>
    <tr>
      <td>build</td>
      <td>MSBuild .targets and .props files</td>
      <td>Automatically inserted into the project file (NuGet 2.x) or project.lock.json (NuGet 3.x).</td>
    </tr>
  </tbody>
</table>

若基于不同版本的Framework需要引用不同版本的dll，可以在lib目录下分别建子目录如net35，net40，net45，之后将所需的dll对应放置

之后在当前布局的父目录，创建.nuspec

        nuget spec




