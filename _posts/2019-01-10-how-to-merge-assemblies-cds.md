---
author: Jan Kostejn
title: >-
  Merging DLLs in the new csproj project format (excluding specific NuGet
  packages)
slug: how-to-merge-assemblies-cds
id: 418
date: '2019-01-10 15:14:24'
categories:
  - Dynamics 365 / CDS / PowerApps
  - English
tags:
  - assemblies
  - Assembly
  - CDS
  - Dynamics
  - dynamics365
  - ILMerge
  - ILRepack
  - merging
  - msbuild
  - new .csproj
---

## TL;DR

There’s no breakthrough in this blogpost… It’s just to show you how does the new .csproj look like, what basic things can be done in msbuild and how-to setup ILRepack for common data service painlessly.

## ILMerge vs. ILRepack

Let me start with this statement: It’s not supported to use ILMerge (or any other tool) for merging assemblies for CDS. But it’s not forbidden either… A lot of you – CDS developers – are using ILMerge. It’s easy to use and works as expected. But it has one flaw. I couldn’t setup ILMerge with the new .csproj format. I think, that it's possible, but this was impulse to move to ILRepack, which I was sure, that it supports the new .csproj format and it’s opensource replacement of ILMerge. [You can check it out on GitHub](https://github.com/gluck/il-repack).

## How to correctly setup ILRepack for your libraries with codeactivities and plugins

This is something that can be done with some research quite easily, there’s nothing hard/special, that you need to know. But I couldn’t find guide that solved all my issues, so I write this blogpost with all steps that I had to apply. I’ll try to explain them, so you can understand how it works. Let’s get to it. Before we start tweaking our .csproj, you need to install ILRepack nuget package to your project. Now it’s just about adding some parts into .csproj:

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project Sdk="Microsoft.NET.Sdk">
<!-- You start with something like this - new .csproj format -->
  <PropertyGroup>
    <TargetFramework>net452</TargetFramework>
    <AssemblyName>Talxis.Buildings.Sales.Features.ARES.Extension.CompositionLayer</AssemblyName>
    <RootNamespace>Talxis.Buildings.Sales.Features.ARES.Extension.CompositionLayer</RootNamespace>
    <Deterministic>false</Deterministic>
    <Version>1.0.0.0</Version>
    <AssemblyVersion>1.0.*</AssemblyVersion>
    <FileVersion>1.0.*</FileVersion>
    <CopyLocalLockFileAssemblies>false</CopyLocalLockFileAssemblies>
  </PropertyGroup>
  <ItemGroup>
    <ProjectReference Include="..\..\..\Infrastructure\Infrastructure.CompositionLayer\Infrastructure.CompositionLayer.csproj" />
    <ProjectReference Include="..\Features.ARES.Extension.BusinessLayer\Sales.Features.ARES.Extension.BusinessLayer.csproj" />
  </ItemGroup>
</Project>
```

<div>

<div>You start with something like this - new .csproj format. Now you can add ILRepack element:</div>

<div>

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net452</TargetFramework>
    <AssemblyName>Talxis.Buildings.Sales.Features.ARES.Extension.CompositionLayer</AssemblyName>
    <RootNamespace>Talxis.Buildings.Sales.Features.ARES.Extension.CompositionLayer</RootNamespace>
    <Deterministic>false</Deterministic>
    <Version>1.0.0.0</Version>
    <AssemblyVersion>1.0.*</AssemblyVersion>
    <FileVersion>1.0.*</FileVersion>
    <CopyLocalLockFileAssemblies>false</CopyLocalLockFileAssemblies>
  </PropertyGroup>
  <ItemGroup>
    <ProjectReference Include="..\..\..\Infrastructure\Infrastructure.CompositionLayer\Infrastructure.CompositionLayer.csproj" />
    <ProjectReference Include="..\Features.ARES.Extension.BusinessLayer\Sales.Features.ARES.Extension.BusinessLayer.csproj" />
  </ItemGroup>

  <!-- You add this Target element for ILRepack -->
  <Target Name="ILRepack" AfterTargets="Build">
    <ItemGroup>
      <ILRepackPackage Include="$(NuGetPackageRoot)\ilrepack\*\tools\ilrepack.exe" />
    </ItemGroup>
    <Error Condition="!Exists(@(ILRepackPackage->'%(FullPath)'))" Text="You are trying to use the ILRepack package, but it is not installed or at the correct location" />
    <Exec Command="@(ILRepackPackage->'%(fullpath)') [<options>] /out:<output assembly> <main assembly> [<other assemblies>]" />
  </Target>
</Project>
```

Replace all values in angle brackets in "Exec" element. My result looks like this:

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net452</TargetFramework>
    <AssemblyName>Talxis.Buildings.Sales.Features.ARES.Extension.CompositionLayer</AssemblyName>
    <RootNamespace>Talxis.Buildings.Sales.Features.ARES.Extension.CompositionLayer</RootNamespace>
    <Deterministic>false</Deterministic>
    <Version>1.0.0.0</Version>
    <AssemblyVersion>1.0.*</AssemblyVersion>
    <FileVersion>1.0.*</FileVersion>
    <CopyLocalLockFileAssemblies>false</CopyLocalLockFileAssemblies>
  </PropertyGroup>
  <ItemGroup>
    <ProjectReference Include="..\..\..\Infrastructure\Infrastructure.CompositionLayer\Infrastructure.CompositionLayer.csproj" />
    <ProjectReference Include="..\Features.ARES.Extension.BusinessLayer\Sales.Features.ARES.Extension.BusinessLayer.csproj" />
  </ItemGroup>

  <!-- You add this Target element for ILRepack -->
  <Target Name="ILRepack" AfterTargets="Build">
    <PropertyGroup>
      <BuildedAssembly>$(OutputPath)$(AssemblyName).dll</BuildedAssembly>
      <!-- Our main assembly -->
    </PropertyGroup>
    <ItemGroup>
      <ILRepackPackage Include="$(NuGetPackageRoot)\ilrepack\*\tools\ilrepack.exe" />
      <OtherAssemblies Include="$(OutputPath)*.dll" Exclude="$(BuildedAssembly)" />
      <!-- All other assemblies in output directory (- referenced projects) -->
    </ItemGroup>
    <Error Condition="!Exists(@(ILRepackPackage->'%(FullPath)'))" Text="You are trying to use the ILRepack package, but it is not installed or at the correct location" />
    <Exec Command="@(ILRepackPackage->'%(fullpath)') /target:library /parallel /keyfile:$(SolutionDir)Talxis.snk ^
          /out:$(BuildedAssembly) $(BuildedAssembly) @(OtherAssemblies,' ')" />
    <!-- CDS registers only signed assemblies, that's why we need to use option /keyfile -->
    <!-- @(OtherAssemblies,' ') prints all other assemblies from output directory separated by space - exactly as ILRepack requires it -->
  </Target>
</Project>
```

<div>

<div>But this won't work, because this would also merge Microsoft.CrmSdk.CoreAssemblie and Microsoft.CrmSdk.Workflow from nuget packages into my main assembly. If we would do this, plugin registration tool wouldn't let us register it (ambiguity).</div>

<div>So we need to reference them, but not merge them. We can achieve this by:</div>

<div>

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net452</TargetFramework>
    <AssemblyName>Talxis.Buildings.Sales.Features.ARES.Extension.CompositionLayer</AssemblyName>
    <RootNamespace>Talxis.Buildings.Sales.Features.ARES.Extension.CompositionLayer</RootNamespace>
    <Deterministic>false</Deterministic>
    <Version>1.0.0.0</Version>
    <AssemblyVersion>1.0.*</AssemblyVersion>
    <FileVersion>1.0.*</FileVersion>
    <CopyLocalLockFileAssemblies>false</CopyLocalLockFileAssemblies>
  </PropertyGroup>
  <ItemGroup>
    <ProjectReference Include="..\..\..\Infrastructure\Infrastructure.CompositionLayer\Infrastructure.CompositionLayer.csproj" />
    <ProjectReference Include="..\Features.ARES.Extension.BusinessLayer\Sales.Features.ARES.Extension.BusinessLayer.csproj" />
  </ItemGroup>

  <!-- This Target element deletes assemblies, that I don't want to be merged -->
  <Target Name="DeleteXRMSDKDlls" AfterTargets="Build">
    <Delete Files="$(OutputPath)\Microsoft.Xrm.Sdk.dll" />
    <Delete Files="$(OutputPath)\Microsoft.Crm.Sdk.Proxy.dll" />
    <Delete Files="$(OutputPath)\Microsoft.Xrm.Sdk.Workflow.dll" />
  </Target>

  <!-- You add this Target element for ILRepack -->
  <Target Name="ILRepack" AfterTargets="DeleteXRMSDKDlls">
    <PropertyGroup>
      <BuildedAssembly>$(OutputPath)$(AssemblyName).dll</BuildedAssembly>
      <!-- Our main assembly -->
    </PropertyGroup>
    <ItemGroup>
      <ILRepackPackage Include="$(NuGetPackageRoot)\ilrepack\*\tools\ilrepack.exe" />
      <OtherAssemblies Include="$(OutputPath)*.dll" Exclude="$(BuildedAssembly)" />
      <!-- All other assemblies in output directory (- referenced projects) -->
    </ItemGroup>
    <Error Condition="!Exists(@(ILRepackPackage->'%(FullPath)'))" Text="You are trying to use the ILRepack package, but it is not installed or at the correct location" />
    <Exec Command="@(ILRepackPackage->'%(fullpath)') /lib:$(SolutionDir)tools\CDSSDKLibraries /target:library /parallel /keyfile:$(SolutionDir)Talxis.snk ^
          /out:$(BuildedAssembly) $(BuildedAssembly) @(OtherAssemblies,' ')" />
    <!-- CDS registers only signed assemblies, that's why we need to use option /keyfile -->
    <!-- @(OtherAssemblies,' ') prints all other assemblies from output directory separated by space - exactly as ILRepack requires it -->
    <!-- But we need to give ILRepack those Microsoft.CrmSdk assmeblies, so it can resolve them - we do that by /lib:<path> option -->
  </Target>

  <!-- This target just cleans other assemblies, since I don't need them -->
  <Target Name="CleanOtherFiles" AfterTargets="ILRepack">
    <ItemGroup>
      <OtherFiles Include="$(OutputPath)*.dll" Exclude="$(OutputPath)$(AssemblyName).dll" />
      <OtherFiles Include="$(OutputPath)*.pdb" />
    </ItemGroup>
    <Delete Files="%(OtherFiles.Identity)" />
  </Target>
</Project>
```

## As I said in the beginning – no breakthrough, but something like this would help me a lot!

</div>

</div>

</div>

</div>