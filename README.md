# Targets
All my targets for visual studio MSBuild, Targets are specific actions which can be applied on your project before or after the build and add some parameters to your visual studio projects.

## Include Unity Dll

The goal of the target is to include Unity Dll in your own project I added only UnityEditor, UnityEngine and UnityEngine.UI. If you use this target you will can create classes wich will use Unity API.

For use it you just have to include the following line in your csproj

```xml
<Import Project="$(PathOfYourTargets)IncludeUnityDll.Targets" 
        Condition="exists('$(PathOfYourTargets)IncludeUnityDll.Targets')" />
```

## Copy files
Copy one or many files in a specific folder, for example I use it for copy my librairies dll in my projects automatically just after the build.

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
  * the property has to be named FilesDestDir. You have to finish it by a \ because the target will consider that.
  
```xml
<FilesDestDir>C:\YourFolderToCopy\</FilesDestDir>
```

  * All that stuff has to be in a PropertyGroup so the full code give :
  
```xml
<PropertyGroup>
  <ItemGroup>
    <MyFiles Include="$(PathOfYourRecursiveFolder)Folder\**\*.*;
              $(PathOfYourFile)MyFile.type"
              Exclude="$(PathOfYourRecursiveFolder)Folder\Folder2\*.*;
              $(PathOfYourFile)Folder\Folder2\MyFile2.type"/>
  </ItemGroup>
  <FilesDestDir>C:\YourFolderToCopy\</FilesDestDir>
</PropertyGroup>
```

* After that you have to call your target, I prefer to call them in the afterbuild target, this is called automatically just after the build.

```xml
<Target Name="AfterBuild">
	<CallTarget Targets="CopyFiles"/>
</Target>
```
I recommend you to create another target and put your PropertyGroup in  and call your target in the after build, for avoid to edit your csproj all the time.

The file will look like :

```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
	<Import Project="$(PathOfYourTargets)CopyFiles.Targets" 
        Condition="exists('$(PathOfYourTargets)CopyFiles.Targets')" />
	<PropertyGroup>
  <ItemGroup>
    <MyFiles Include="$(PathOfYourRecursiveFolder)Folder\**\*.*;
              $(PathOfYourFile)MyFile.type"
              Exclude="$(PathOfYourRecursiveFolder)Folder\Folder2\*.*;
              $(PathOfYourFile)Folder\Folder2\MyFile2.type"/>
  </ItemGroup>
  <FilesDestDir>C:\YourFolderToCopy\</FilesDestDir>
</PropertyGroup>
	<Target Name="CopyFilesByMYTarget">
		<CallTarget Targets="CopyFiles"/>
	</Target>
</Project>
```

and in your csproj you will just have :

```xml
<Import Project="$(PathOfYourTargets)CopyFilesByMYTarget.Targets" 
        Condition="exists('$(PathOfYourTargets)CopyFilesByMYTarget.Targets')" />
<Target Name="AfterBuild">
	<CallTarget Targets="CopyFilesByMYTarget"/>
</Target>
```

## Copy scripts

Copy all your C# scripts in a folder. I could used the copy files but this target say when it copy interfaces and when it copy classes and I like know this things. Also I don't have to specify wich files to copy because it will search Interfaces and Classes folders in your own project folder (where you find the csproj).  It can be usefull for create a save of your scripts.
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
* Finally call it in your csproj (I used the afterbuild too for being sure my scripts build with success)

```xml
<Target Name="AfterBuild">
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
	<Target Name="CopyScriptsByMyTarget">
		<CallTarget Targets="CopyScripts"/>
	</Target>
</Project>
```

and in the csproj :

```xml
<Import Project="$(PathOfYourTargets)CopyScriptsByMyTarget.Targets" 
        Condition="exists('$(PathOfYourTargets)CopyScriptsByMyTarget.Targets')" />
<Target Name="AfterBuild">
	<CallTarget Targets="CopyScriptsByMyTarget"/>
</Target
```  
___________________________________

That's all for now enjoy :-D
