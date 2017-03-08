---
layout: post
title: Deploying WebSites with MSDeploy and MSBuild
---

At the end of this text you'll discover how to build packages from source files and how to deploy them to a webserver.

# MSBuild

In Microsofts [own words](https://msdn.microsoft.com/en-us/library/dd393574.aspx) 

> The Microsoft Build Engine is a platform for building applications. This engine, which is also known as MSBuild, provides an **XML schema** for a project file that controls how the build platform processes and builds software. Visual Studio uses MSBuild, but it doesn't depend on Visual Studio. By invoking msbuild.exe on your project or solution file, you can orchestrate and build products in environments where Visual Studio isn't installed.

The two ways build tools differ are task orient vs. product-oriented. Task oriented tools describe the dependency of networks in terms of a specific set task and product-oriented tools describe things in terms of the products they generate.

## Install MSBuild

MSBuild used be delivered as part of the .NET Framework. Since Visual Studio 2013 it now also [comes bundled](https://blogs.msdn.microsoft.com/visualstudio/2013/07/24/msbuild-is-now-part-of-visual-studio/) with the IDE. If you don't already have MSBuild, please download the standalone [Microsoft Build Tools 2015](https://www.microsoft.com/en-us/download/details.aspx?id=48159). You should now be able to find msbuild.exe and csc.exe (C# Compiler) in **C:\Program Files (x86)\MSBuild\14.0\Bin** and add it to your path.

### MSBuild Project file

### Create a minimal Project file

This chapter is adapting Microsoft's [Creating an MSBuild Project File from Scratch](https://msdn.microsoft.com/en-us/library/dd576348.aspx). Let's create a minimal MSBuild project file named foo.build (the name does not really matter) with the contents:

```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Target Name="SampleTarget1">
    <Message Text="This is a target, it does something" />
  </Target>
  
  <Target Name="SampleTarget2">
    <Message Text="This is a another target, it does something else" />
  </Target>

</Project>
```

If we pass this file to msbuild `msbuild.exe foo.build`, we'll see that SampleTarget1 was executed. This is due to the MSBuild [Target Build Order](https://msdn.microsoft.com/en-us/library/ee216359.aspx):

> If there are no initial targets, default targets, or command-line targets, then MSBuild runs the first target it encounters in the project file or any imported project files.

We can force MSBuild to execute SampleTarget2 by passing it via command-line `msbuild.exe foo.build /target:SampleTarget2` or by specifying the default target in the project file:

```xml
<Project DefaultTargets="SampleTarget1;SampleTarget2" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
...
```

With DependsOnTargets, BeforeTargets or AfterTargets you can specify the target order in your project file.

### Compile Code with MSBuild

Let's create a HelloWorld.cs file which we can compile:

```cs
using System;  

class HelloWorld  
{  
    static void Main()  
    {  
        Console.WriteLine("Hello, world!");  
    }  
}

```

We can compile this file with our c# compiler if we want `csc.exe HelloWorld.cs`, which will result in HelloWorld.exe. If we have thousends of files that might become annoying, so we'll use our project file instead:

```xml
<Project DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <ItemGroup>  
    <Compile Include="HelloWorld.cs" />  
  </ItemGroup>  

  <Target Name="Build">  
    <Csc Sources="@(Compile)"/>    
  </Target>  
</Project> 
```

If we run msbuild again `msbuild.exe foo.build` we'll get the same HelloWorld.exe. If we want to we can also change the name of our assembly:

```xml
<Project DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <OutputAssembly>HiWorld</OutputAssembly>
  </PropertyGroup>

  <ItemGroup>  
    <Compile Include="HelloWorld.cs" />  
  </ItemGroup>  
  
  <Target Name="Build">  
    <Csc 
      Sources="@(Compile)"
      OutputAssembly="$(OutputAssembly).exe"/>    
  </Target>  
</Project> 
```

In our project file we can already see several different elements. We have  

* [Properties](https://msdn.microsoft.com/en-us/library/ms171458(v=vs.140).aspx), which are key-value pairs and are accessed via $(PropertyName)
* [Items](https://msdn.microsoft.com/en-us/library/ms171453.aspx), which act like a list, are usually used for files and are expanded into lists using @(ItemName)
* [Targets](https://msdn.microsoft.com/en-us/library/ms171462.aspx) that group tasks together in a particular order
* [Tasks](https://msdn.microsoft.com/en-us/library/ms171466.aspx) like [CSC which wraps CSC.exe](https://msdn.microsoft.com/en-us/library/s5c8athz.aspx), which is executable code that performs an atomic build operation

### MSBuild Items
MSBuild items are inputs into the build system, and they typically represent files.


```xml
<ItemGroup>  
    <Compile Include = "file1.cs;file2.cs"/>  
</ItemGroup> 
```

is equivalent to:

```xml
<ItemGroup>  
    <Compile Include = "file1.cs"/>  
    <Compile Include = "file2.cs"/>   
</ItemGroup>
```

Please note that:
> The item "file2.cs" doesn’t replace the item "file1.cs"; instead, the file name is appended to the list of values for the Compile item type. You can’t remove an item from an item type during the evaluation phase of a build.

We can use @(ItemName) to expand the ItemGroup into a semicolon delimited list.

Items can have meta-data (in the form of key-value pairs) and you can reference that metadata using _%(ItemMetadataName)_ or not-as-ambigious _%(ItemType.ItemMetaDataName)_.

```xml
<ItemGroup>  
    <Stuff Include="One.cs" >  
        <Display>false</Display>  
    </Stuff>  
    <Stuff Include="Two.cs">  
        <Display>true</Display>  
    </Stuff>  
</ItemGroup>  
<Target Name="Batching">  
    <Message Text="@(Stuff)" Condition=" '%(Display)' == 'true' "/>  
</Target>  
```

> You can transform item lists into new item lists by using metadata. For example, you can transform an item type CppFiles that has items that represent .cpp files into a corresponding list of .obj files by using the expression @(CppFiles -> '%(Filename).obj').

### MSBuild Properties
Properties are useful for passing values to tasks, evaluating conditions, and storing values that will be referenced throughout the project file. Properties are referenced by using the syntax $(PropertyName). Since lists are just semicolon deliminated lists, we can also use properties to represent multiple values (see [Comparing Properties and Items](https://msdn.microsoft.com/en-us/library/dd997067.aspx)):

```xml
<PropertyGroup>  
    <BuildDependsOn>  
        BeforeBuild;  
        CoreBuild;  
        AfterBuild  
    </BuildDependsOn>  
</PropertyGroup> 
```

Properties are evaluated in the order in which they appear in the project file. That means their values can be changed by redefining the property.

MSBuild reserves [property names like $(MSBuildBinPath)](https://msdn.microsoft.com/de-de/library/bb629394.aspx) to store information about the project file and the MSBuild binaries. It also makes many [macros like `$(ProjectDir)](https://msdn.microsoft.com/en-us/library/c02as0cs.aspx) ` available to you. 

Starting in .NET Framework version 4, you can use property functions to evaluate your MSBuild scripts `<Today>$([System.DateTime]::Now.ToString("yyyy.MM.dd"))</Today>`

### A less redundant project file



`<Import Project="$(MSBuildToolsPath)\Microsoft.CSharp.targets" /> `



# MSDeploy

asds

# Packaging and Deploying a Website

asdas

# Packaging and Deploying a Console Application

asdas