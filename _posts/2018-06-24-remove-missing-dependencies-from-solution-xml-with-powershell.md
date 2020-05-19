---
author: Tomas Prokop
title: Remove missing dependencies from solution XML with PowerShell
slug: remove-missing-dependencies-from-solution-xml-with-powershell
id: 191
date: '2018-06-24 20:55:34'
categories:
  - Dynamics 365 / CDS / PowerApps
tags:
  - CRM
  - Dynamics 365
  - Import
  - PowerShell
  - Solution
  - XML
---

_The import of the solution XYZ failed. The following components are missing in your system and are not included in the solution. Import the managed solutions that contain these components (Active) and then try importing this solution again._ If you ever run into this exception and there are all the components already present in the environment you just need to get rid of few lines in a solution definition in the ZIP file you are trying to import. **Do this only if you are absolutely sure that you know what you are doing.** This is caused by this section of solution.xml file which is inside your solution's ZIP file. ![](/uploads/2018/06/explorer_2018-06-24_21-40-05.png)   ![](/uploads/2018/06/Code_2018-06-24_21-42-02.png)   The import wizard does not perform any check whether the component is actually present in the environment. ![](/uploads/2018/06/solutionimport.png) There may be a situation when you have all the components in place and solution import should proceed without issues but the wizard throws this error. In this case you can delete dependencies from the definition. If you do it quite often or you need to make it part of your automated deployment, here is a PowerShell script just for that:

```powershell
param (
    [string]$zipfileName
)

$fileToEdit = "solution.xml"
$zipfileName = Resolve-Path $zipfileName

# Open zip and find the solution.xml file
Add-Type -assembly  System.IO.Compression.FileSystem
$zip =  [System.IO.Compression.ZipFile]::Open($zipfileName,"Update")
$solutionFile = $zip.Entries.Where({$_.name -eq $fileToEdit})

# Read the XML
$streamReader = [System.IO.StreamReader]($solutionFile).Open()
$XmlDocument = [xml]$streamReader.ReadToEnd()
$XmlDocument.PreserveWhiteSpace = $true
$streamReader.Close()

# Remove MissingDependency nodes
if ($XmlDocument.ImportExportXml.SolutionManifest.MissingDependencies -is [Xml.XmlElement]) {
    $XmlDocument.ImportExportXml.SolutionManifest.MissingDependencies.MissingDependency | %{ $_.ParentNode.RemoveChild($_) | Out-Null }
}

# Overwrite the file
$streamWriter = [System.IO.StreamWriter]($solutionFile).Open()
$streamWriter.BaseStream.SetLength(0)
$streamWriter.Write($XmlDocument.OuterXml)
$streamWriter.Flush()
$streamWriter.Close()

# Close the zip file
$zip.Dispose()
```

  <span style="text-decoration: underline;">You can use it like this:</span>

1.  Save the script in a new file. Call it RemoveMissingDependencies.ps1 and place it to the same folder where you're downloading your solutions.
2.  Open PowerShell in the folder where you downloaded your solution's ZIP file and where you placed your script. ![](/uploads/2018/06/2018-06-24_21-47-33.png)
3.  Invoke this: **.\RemoveMissingDependencies.ps1 YourSolution_1_0_0_0_managed.zip ![](/uploads/2018/06/powershell_2018-06-24_21-52-59.png)**
4.  Now you can import the modified file.

  UPDATE 7/6/2018: The function is included in [Dynamics 365 Release Automation Tools](https://github.com/TheNetworg/dynamics365-release-automation-tools)