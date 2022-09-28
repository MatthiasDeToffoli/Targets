# Targets
This repository contain all my targets for visual studio MSBuild.

Targets are specific actions which can be applied on your project before, after or instead of the build and add some parameters to your visual studio projects.

for use a target you just have to import it in your csproj. 
For that, write :
```xml
<Import Project="$(FullPathOfYourTargets)" Condition="exists('$(FullPathOfYourTargets)')" />
```
some targets has to be called you have many way to call them in the order you want. Me, I use three things for now, I will describe them quickly :
at the declaration of the csproj (or the targets file) you can change the default target :
```xml
<Project Sdk="Microsoft.NET.Sdk" DefaultTargets="MyTarget">
```
if you don't call the build, the project will not do it (and not create your dlls for exemple) .
If you don't write de DefaultTargets then the default one will be Build

you can add mutliple target as default like :
```xml
<Project Sdk="Microsoft.NET.Sdk" DefaultTargets="MyTarget;Build;MyTarget2">
```
It will call all the targets in the order write but me I don't like that, that's why I use two others attributs.

You have to set the two others when you declare a target, for declare a target you have to write :

```xml
<Target Name="MyTarget">
	<!-- Do something... -->
</Target>
```
everything you write instead of the "do something" will be call when you will call your target. So for call it before the build use the attribut BeforeTargets :
```xml
<!-- this target is call before the build -->
<Target Name="MyTarget" BeforeTargets="Build">
	<!-- Do something... -->
</Target>
```

if you didn't specify the default target so it will call your target and after it the build. 

The other I use is AfterTargets I think you understand that mean your target will be called after all others targets :

```xml
<!-- this target is call after the build -->
<Target Name="MyTarget2" AfterTargets="Build">
	<!-- Do something... -->
</Target>
```

you can also add multiple targets in the two attributes :
```xml
<!-- this target is call after MyTarget1 and Build and before MyTarget2 and MyTarget4 the build -->
<Target Name="MyTarget3" BeforeTargets="MyTarget2; MyTarget4" AfterTargets="MyTarget1; Build">
	<!-- Do something... -->
</Target>
```
## Include Unity Dll

The goal of the target is to include Unity Dll in your own project I added only UnityEditor, UnityEngine, UnityEngine.IMGUIModule, InputLegacyModule, UnityEngine.CoreModule, UnityEngine.TextRenderingModule and UnityEngine.UI. If you use this target you will can create classes wich will use Unity API.

For use it you just have to include the following line in your csproj

```xml
<Import Project="$(PathOfYourTargets)IncludeUnityDll.Targets" 
        Condition="exists('$(PathOfYourTargets)IncludeUnityDll.Targets')" />
```
the paths of the dlls included is the ones of Unity 2019.3.13f1

## Copy files
Copy one or many files in a specific folder, for example I use it for copy my librairies dlls in my projects automatically just after the build.

Using it can be a little more complicate.

* First you have to include the target in your csproj
```xml
<Import Project="$(PathOfYourTargets)CopyFiles.Targets" 
        Condition="exists('$(PathOfYourTargets)CopyFiles.Targets')" />
```
* After that you have to create an Item including all your files to copy and a property for specify the folder where you want to copy your files.
  * Your item has to be named MyFiles you can add files or folders which can be recursive. Use include and exclude for say what will be copied and what will not be. An item as to be in a ItemGroup
  
```xml
<ItemGroup>
  <MyFiles Include="$(PathOfYourRecursiveFolder)Folder\**\*.*;
          $(PathOfYourFile)MyFile.type"
          Exclude="$(PathOfYourRecursiveFolder)Folder\Folder2\*.*;
          $(PathOfYourFile)Folder\Folder2\MyFile2.type"/>
</ItemGroup>
```
  * the property has to be named FilesDestDir. You have to finish it by a \ because the target will consider that. This property has to be in a propertygroup
  
```xml
<PropertyGroup>
	<FilesDestDir>C:\YourFolderToCopy\</FilesDestDir>
</PropertyGroup>
```

* After that you have to call your target :

```xml
<Target Name="MyTarget" AfterTargets="Build">
	<CallTarget Targets="CopyFiles"/>
</Target>
```
I recommend you to create another target and put your PropertyGroup in  and call your target, for avoid to edit your csproj all the time.

The file will look like :

```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
	<Import Project="$(PathOfYourTargets)CopyFiles.Targets" 
        	Condition="exists('$(PathOfYourTargets)CopyFiles.Targets')" />
  	<ItemGroup>
	    <MyFiles Include="$(PathOfYourRecursiveFolder)Folder\**\*.*;
		      $(PathOfYourFile)MyFile.type"
		      Exclude="$(PathOfYourRecursiveFolder)Folder\Folder2\*.*;
		      $(PathOfYourFile)Folder\Folder2\MyFile2.type"/>
  	</ItemGroup>
	<PropertyGroup>
  		<FilesDestDir>C:\YourFolderToCopy\</FilesDestDir>
	</PropertyGroup>
	<Target Name="CopyFilesByMYTarget" AfterTargets="Build">
		<CallTarget Targets="CopyFiles"/>
	</Target>
</Project>
```

and in your csproj you will just have :

```xml
<Import Project="$(PathOfYourTargets)CopyFilesByMYTarget.Targets" 
        Condition="exists('$(PathOfYourTargets)CopyFilesByMYTarget.Targets')" />
```

## Copy scripts

Copy all your C# scripts in a folder. I could used the copy files but this target say when it copy interfaces and when it copy classes and I like know this things. Also I don't have to specify wich files to copy because it will search Interfaces and Classes folders in your own project folder (where you find the csproj).  It can be usefull for use your scripts in other projects without use a dll.
First of all the target will delete the folders Interfaces and Classes in the Output folder for clear all file which not exist in the input folder.
It's more easy than the previous to use because you will not have to specify what files to take, it take all files recurcively in Classes and Interfaces.

* As always start by include the target

```xml
<Import Project="$(PathOfYourTargets)CopyScripts.Targets" 
        Condition="exists('$(PathOfYourTargets)CopyScripts.Targets')" />
```
* After that create the property for set the destination folder (the property has to be named DestScriptsDir and to finish with a \)

```xml
<PropertyGroup>
  <DestScriptsDir>C:\YourFolderToCopy\</DestScriptsDir>
</PropertyGroup>
```
* Finally call it in your csproj (I call it after the build for being sure my scripts build with success)

```xml
<Target Name="MyTarget" AfterTargets="Build">
	<CallTarget Targets="CopyScripts"/>
</Target>
```

I encourage you to use your own target too, it will look like :

```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
	<Import Project="$(PathOfYourTargets)CopyScripts.Targets" 
        	Condition="exists('$(PathOfYourTargets)CopyScripts.Targets')" />
	<PropertyGroup>
    		<DestScriptsDir>C:\YourFolderToCopy</DestScriptsDir>
	</PropertyGroup>
	<Target Name="CopyScriptsByMyTarget" AfterTargets="Build">
		<CallTarget Targets="CopyScripts"/>
	</Target>
</Project>
```

and in the csproj :

```xml
<Import Project="$(PathOfYourTargets)CopyScriptsByMyTarget.Targets" 
        Condition="exists('$(PathOfYourTargets)CopyScriptsByMyTarget.Targets')" />
```  
___

*<sub>Made with Unity 2021.3.10f1 , Visual studio Community 2022 version 17.2.1 and .Net Standard 2.1</sub>*
