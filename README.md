
# **H**ollow **K**night Mod **B**uild **Utils**

 [![Build](https://github.com/HKLab/HKBuildUtils/actions/workflows/dotnet.yml/badge.svg)](https://github.com/HKLab/HKBuildUtils/actions/workflows/dotnet.yml) ![Nuget](https://img.shields.io/nuget/v/HKBuildUtils)

A nuget package to help make the Hollow Knight Mod

## Features

### Automatically Pack Mods

Automatically pack the mod into a zip file and calculate the SHA256 value after the build.

#### Configuration

- `$(ExportDir)`: Specifies the directory for the packaged output. Defaults to "$(ProjectDir)\bin\Publish\"
- `$(PublishZipPath)`: Specify the save path of the ZIP output after packaging. Defaults to "$(ExportDir)\Publish.zip"
- `$(ReleaseInfoPath)`: Specifies the output location of the ReleaseInfo information constructed after packaging. Currently, only SHA256 is included. Default is "$([System.IO.Path]::GetDirectoryName('$(PublishZipPath)'))\ReleaseInfo.txt"

### Mods References Helper

Automatically download HKMAPI and other mods that depend on it when building.

#### How to use

Add the names of the mods you want to depend on to the `<ItemGroup>` in the following way

```xml
<ModReference Include="<Mod Name in ModLinks>" AssemblyName="[Assembly Name]" />
```

Among them, `AssemblyName` is optional, but due to the limitation of MSBuild, if the Mod assembly file name does not match its name on ModLinks, you need to fill in `AssemblyName` by yourself.

For example, Custom Knight, its assembly file name is `CustomKnight.dll` instead of `Custom Knight.dll`, therefore, it needs to be referenced as follows

```xml
<ModReference Include="Custom Knight" AssemblyName="CustomKnight" />
```

#### Set game path

Set game paths to use game files directly instead of downloading them again.

HKBuildUtils will look for the game directory in the following order

(1.) The contents of hkpath.txt under the project folder or its ancestor folder

(2.) `HollowKnightRefs` defined in the project file

> Note:
> 1. (1.) will be used if both (1.) and (2.) are found, you can disable (1.) and use (2.) by using `<DisableOverwriteHollowKnightRefs>true</DisableOverwriteHollowKnightRefs>`
> 2. If neither of them exists or the directory they specify does not exist, it will be considered as a build in CI, and the dependent mods and HKMAPI will be downloaded to the `~ModLibrary` folder under the project folder during the build


### Reflect Helper

Used to replace the Reflect Helper of HKMirror

Allows developers to directly access private fields and private types

**If you need to use this feature, please add a reference to the Fody package**

Use `<ReflectHelper Include="<AssemblyName>" />` to indicate which assemblies need to construct Reflect Helper

> `<AssemblyName>` can use assemblies explicitly referenced in project files using `<Reference>` and MAPI assemblies and other Mods assemblies inserted by HKBU

For example:

```xml
<ReflectHelper Include="UnityEngine.CoreModule" />
<ReflectHelper Include="Assembly-CSharp" />
```

They indicate the Reflect Helper that generates UnityEngine.CoreModule and Assembly-CSharp

They can be used like

```c#
HeroControllerR reflect = HeroController.instance.Reflect(); 
//or HeroControllerR reflect = (HeroControllerR)HeroController.instance;

Log(reflect.rb2d.name);

ModLoaderR.TryAddModInstance(typeof(TestModMod), new ModInstanceR()
{
   Enabled = true,
   Mod = this,
   Name = "Hello, World!This is Test Mod"
}); 
```

#### Compare with HKMirror

Advantage:

- No need to add additional references
- Can be used with almost any assembly
- Has most of the private types
- Custom operator operations with primitive types
- Primitive object instances can be created directly using `new`. For example, `new ModInstanceR()` will create an instance of `ModLoader.ModInstance`
- Supports generics


### Mono Mod Hooks Helper

Allow developers to use MonoModHooks outside of `MMHOOK_Assembly-CSharp` and `MMHOOK_PlayMaker.dll`

They are automatically generated and automatically referenced at build

**If you need to use this feature, please add a reference to the Fody package**

Use `<MonoModHook Include="<AssemblyName>" />` to indicate which assemblies need to construct MonoModHookHelper

> `<AssemblyName>` can use assemblies explicitly referenced in project files using `<Reference>` and MAPI assemblies and other Mods assemblies inserted by HKBU

For example:

```xml
<MonoModHook Include="UnityEngine.CoreModule" />
<MonoModHook Include="Assembly-CSharp" />
```

They indicate the MonoHookHelper that generates UnityEngine.CoreModule and Assembly-CSharp

To avoid errors, this automatically disables the reference to MMHOOK_Assembly-CSharp that comes with MAPI

MonoModHookHelper (including those that come with MAPI) will be merged into the mod assembly by Fody in the build, no need to publish additional files

### Merger with HKMirror

Use ILRepack to merge mods with [HKMirror](https://github.com/TheMulhima/HKMirror) and eliminate unused types in [HKMirror](https://github.com/TheMulhima/HKMirror).

> It is recommended to use [Reflect Helper](#Reflect-Helper)

#### How to enable

Add `<MergeHKMirror>true</MergeHKMirror>` to `PropertyGroup` and reference [HKMirror](https://github.com/TheMulhima/HKMirror).
The rest will take care of it for you.

### Mod Resources

Using `<ModResource></ModResource>` instead of `<EmbeddedResource></EmbeddedResource>` automatically generates the type `ModResources` for use

#### Example

Add the following to the project file

```xml
<ItemGroup>
    <ModResource Include="Test1.txt"></ModResource>
    <ModResource Include="a/Test2.txt"></ModResource>
    <ModResource Include="b/c/Test3.txt"></ModResource>
</ItemGroup>
```

Then you can get them like this in your code

```c#
Modding.Logger.Log(ModResources.TEST1.Length);

Modding.Logger.Log(ModResources.TEST2.Length);

Modding.Logger.Log(ModResources.TEST3.Length);
```

You can add `Default="true"` to `<ModResource>` to use user-defined content.Like this

```xml
<ItemGroup>
    <ModResource Include="a/b/c" Default="true"></ModResource>
</ItemGroup>
```

You can get it through `ModResources.C`, just like mentioned above.

If `ModResources.C` is used, the file `$(ModDir)/a/b/c` is checked for existence first. If present, the file contents are returned, and if they are not, read from the assembly and written to the `$(ModDir)/a/b/c` file
