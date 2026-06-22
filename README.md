# TS-Build
This is a automated Build script setup you can use to automatically package your mods for [Thunderstore](https://thunderstore.io). This uses `dotnet publish` to build the package.

This is released as Open-Source subject to the terms found within [SML-1.0.0](https://stoatgames.icu/license-repository/licenses/sml-1-0-0/#content)

## Credits:
* Chaosyr: Creation of the `TS-Build.bat`.
* Cecil Libraries Organization: The Original Version of the Build Script, in which we will be fully integrating down the line with the help of their Organization.

## What is this built for?

This script is built for C# mods for Thunderstore meaning that if your mods are any other language your unfortunately out of luck. For Java mods I reccomend using BlueAzures [HytalePublisher](https://github.com/AzureDoom/HytalePublisher). Do note its built primarily for Hytale mods, so you might have a better time forking this repository and modifying elements for where properties are fetched from.

## What does it do?

This package essentially builds your mod, fetches you README, CHANGELOG, LICENSE, Manifest Fields, and any folders you wish to copy over into automatically built zips targetted for the Thunderstore ecosystem.

## How can I set this up?

1. First you'll want to download the `.bat` file included in this repository and any other build script that may be present entitled `TS-Build`.
2. Drop it into your CSProj's Root.
3. Open the file in a text editor and modify the value of `CSProj` to `%~dp0{YOUR CSPROJ'S NAME}.csproj` where `{YOUR CSPROJ'S NAME}` is the name of the literal file that holds the CSProj you wish to target, excluding the extension. The `%~dp0` handles getting the full path based on where the bat file is. So as long as the bat's where it should be you don't need to worry about paths.
4. After changing the value hit save and touch nothing else.

### Configuration

This project uses several `.csproj` fields to configure how the script works, below is a table of CSProj Keys and what their values should be.

|              `.CSProj` Key               |                                                                                                                                    `.CSProj` Value(s)                                                                                                                                    |
|:----------------------------------------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
|           `<TargetFrameworks>`           |                                              This should be a list `;` seperated of all netframeworks the mod works with, or just the primary one, No `<TargetFramework>` does not work with this. If its invalid the fallback is `net6.0`.                                              |
|          `<RuntimeIdentifiers>`          |                                                                  This should be a list `;` seperated of all Runtime Identifiers the mod works with, or just the primary one. If its invalid the fallback is `win-x64`.                                                                   |
|          `<BuildCopyLocations>`          |             This should be a `;` seperated list of all paths (root localized) in which should be copied directly to the package, keep in mind by directly we mean its root folder, not the plugins subfolder. If its invalid the fallback is `hunderstore-Package-Elements`.             |
|        `<CHANGELOGCopyLocation>`         |                                                                                                                    The root localized path to the Changelog location.                                                                                                                    |
|          `<READMECopyLocation>`          |                                                                                                                     The root localized path to the README location.                                                                                                                      |
|         `<LICENSECopyLocation>`          |                                                                                                                     The root localized path to the LICENSE location.                                                                                                                     |
|      `<DependenciesCopyLocations>`       | The root localized path to the Dependencies location. Note these should be dependencies needed by your mod to function that aren't covered by the Mod Installation setup itself. For example NuGet packages. If its invalid the fallback is `Dependencies/BepInEx-Required-Dependencies` |
|             `<ReleaseMode>`              |                                                      A string with the value of `Release` for Release Publishing or `Debug` for Debug Publishing (otherwise known as Prerelease). If the value is invalid the fallback is `Debug`.                                                       |
|       `<ThunderstorePackageName>`        |                                                                           The Thunderstore Package Name. It must adhere to the following spec: Name of the mod, no spaces. Allowed characters: `a-z A-Z 0-9 _`                                                                           |
|      `<ThunderstorePackageVersion>`      |                                                                The Thunderstore Package Version. It must adhere to the following spec: Version number of the mod, following the semantic version format Major.Minor.Patch                                                                |
|        `<ThunderstoreWebsiteURL>`        |                                                          The Website URL in which your package should link to. It must adhere to the following spec: URL of the mod's website (e.g. GitHub repo). Can be left an empty string.                                                           |
|    `<ThunderstorePackageDescription>`    |                                                                 The short description of your Package. It must adhere to the following spec: A short description of the mod, shown on the mod list. Max 250 characters.                                                                  |
| `<ThunderstorePackageDependencyStrings>` |                                     This should be a `;` seperated list of Dependency strings, you can get the package string by visiting the given dependency page on https://thunderstore.io and copying the field value of: `Dependency string`.                                      |

All fields should be treated as required.

An example setup is:

```xml
    <!--Build System Properties-->
    <PropertyGroup>
        <!--Copy Locations-->
        <BuildCopyLocations>Thunderstore-Package-Elements</BuildCopyLocations>
        <CHANGELOGCopyLocation>CHANGELOG.md</CHANGELOGCopyLocation>
        <READMECopyLocation>README.md</READMECopyLocation>
        <LICENSECopyLocation>LICENSE.md</LICENSECopyLocation>
        <DependenciesCopyLocations>Dependencies/BepInEx-Required-Dependencies</DependenciesCopyLocations>
        <ReleaseMode>Release</ReleaseMode>
        <!--Thunderstore Manifest Properties-->
        <ThunderstorePackageName>JSONCardLoader</ThunderstorePackageName>
        <ThunderstorePackageVersion>2.7.0</ThunderstorePackageVersion>
        <ThunderstoreWebsiteURL>https://github.com/MADH95/JSONCardLoaderPlugin</ThunderstoreWebsiteURL>
        <ThunderstorePackageDescription>A mod made for Incryption to create custom cards, sigils, starter decks, tribes, encounters and more using JSON files.</ThunderstorePackageDescription>
        <ThunderstorePackageDependencyStrings>BepInEx-BepInExPack_Inscryption-5.4.1902;API_dev-API-2.23.7</ThunderstorePackageDependencyStrings>
    </PropertyGroup>
```

Note that `<TargetFrameworks>` and `<RuntimeIdentifiers>` are not included in this block, that is because they are project wide fields thus they should be defined at the top of your `.csproj` file.

### Build Cleaning

This system does not have a clean way of cleaning up the build output on its own, thats why we have a couple more properties in our `.csproj` system that come directly from the MSBuild system to help resolve this.

First add these 3 properties to the top section of your `.csproj` 

* `SelfContained` will make it so your executable mod is able to run on its own
* `CopyLocalLockFileAssemblies` will make prevent local assemblies from being included in output when publishing or building your project (our build system handles this on its own via the configured property above)
* `IncludeBuildOutput` will make it so that BuildOutput itself won't be included when publishing (we only want the Assemblies we need).

```xml
        <SelfContained>true</SelfContained>
        <CopyLocalLockFileAssemblies>false</CopyLocalLockFileAssemblies>
        <IncludeBuildOutput>false</IncludeBuildOutput>
```

With Local Dependencies, if you want them to copy to the build folder add the following after referencing them in your `.csproj`

```xml
        <Private>true</Private>
```

This should appear in the reference as follows:

```xml
        <Reference Include="Antlr3.Runtime">
            <HintPath>Dependencies\BepInEx-Required-Dependencies\Antlr3.Runtime.dll</HintPath>
            <Private>true</Private>
        </Reference>
```

Another item we have to help with Cleanup is the following Target:

```
    <!--Cleanup Publish for our Build System-->
    <Target Name="StripAllNuGetFromPublish" AfterTargets="ComputeFilesToPublish">
        <ItemGroup>
            <ResolvedFileToPublish Remove="@(ResolvedFileToPublish)" Condition="'%(ResolvedFileToPublish.Extension)' == '.dll' and '%(ResolvedFileToPublish.RelativePath)' != 'JSONLoader.dll'" />
        </ItemGroup>
    </Target>
```

This makes it so the Output will remove anything that is not `JSONLoader` in this case, Replace `JSONLoader` with the value of your `.csproj`'s `AssemblyName` field.

## Running

When it comes to running you can either launch the script via Terminal (ideally Powershell) or you can do the following to setup a runner for your Mod. For launching via Terminal make sure to navigate to the folder `TS-Build.bat` resides in first before running it!

### JetBrains Rider

1. Press `Add New Configuration` or the Arrow next to the Configuration already selected.
2. Press `Edit Configuraion`.
3. Press the `+` icon than select `Native Executable`.
4. On the field `Executable Path` click the folder icon (Browse) and navigate to the `TS-Build.bat` and select the file.
5. If it was set properly the `Exe Path` and `Working Directory` should have updated.
6. Hit `Store As Project File`, this will add a file under `.run` the next time you load the project.
7. Give it a Name, than hit `Apply` followed by `Okay`.
8. Select the Runner you just created and hit `Run`. (Play Button)
9. If everythings working it should finish the entire build, check the output directory and see if it matches what you were after, if it doesn't its likely you either misconfigured something or didn't using a correct pathing scheme.

### Universally

1. In the Solution folder create a new folder called `.run`.
2. Open that folder and create a new file called `TS-Build.run.xml`
3. Paste in the following codeblock
    ```xml
    <component name="ProjectRunConfigurationManager">
      <configuration default="false" name="TS-Build" type="RunNativeExe"    factoryName="Native Executable">
        <option name="EXE_PATH" value="$PROJECT_DIR$/{PROJECT FOLDER}/TS-Build.bat" />
        <option name="PROGRAM_PARAMETERS" value="" />
        <option name="WORKING_DIRECTORY" value="$PROJECT_DIR$/{PROJECT FOLDER}" />
        <option name="PASS_PARENT_ENVS" value="1" />
        <option name="ENV_FILE_PATHS" value="" />
        <option name="REDIRECT_INPUT_PATH" value="" />
        <option name="MIXED_MODE_DEBUG" value="0" />
        <method v="2" />
      </configuration>
    </component>
    ```
4. Replace both instances of `{PROJECT FOLDER}` with the folder path leading to your Project for example `JSONLoaderOld`. `$PROJECT_DIR$` will resolve all the way up to the Solution folder, you just need to add the rest of the path.
5. Save the file and do not touch anything else.
6. Within your IDE if its supported you should see `TS-Build` as a runner.

## Where do Builds go?

They go either into a folder called `GITHUB RELEASE` or `GITHUB PRERELEASE` dependant of your configuration. Both folders are found in the root location of this script.

## How is Output Structured?

There will be 3 subfolders (2 of these NYI with the current TS-Build System but they will be available in the future)
* `NET`: this is where any directive NET packages will resolve to.
* `NATIVE`: this is where any NativeAOT build packages will resolve to.
* `NUGET`: this is where any NUGET Packages will resolve to.

All packages will appear in the folowing formats:

* `NET`: `{target-framework}-{runtime-identifier}-{release-mode}.zip`
* `NATIVE`: `Native-{target-framework}-{runtime-identifier}-{release-mode}.zip`
* `NUGET`: The typical `dotnet pack` output.

## How you can support our Projects and their Developers:

* You can donate to Chaosyr (The Primary Developer in this case) here:
  * Use code `Chaosyr` in Hytale Checkout, they will receive a portion of the Checkout proceeds.
  * Use their KO-FI and donate or commission: https://ko-fi.com/thincreator3483
* You can help Financially Support Stoat Games here:
  * Use our affiliate link with IONOS: https://aklam.io/thGxWQTD (This is the most powerful method for keeping The Docs site alive.)
  * Use our affiliate link with The Free Website Guys: https://thefreewebsiteguys.com/?js=a16638001b (Less powerful but it will support The Docs Site, and them)
  * When the Option opens you can choose US to host your Projects Docs or Your Organizations Subsite under [Stoat Games](https://stoatgames.icu)
* Contribute to our Projects
  * You can send Pull Requests to any of our Projects to help make them work better for future users.
  * You can submit Bug Reports so that we can help More Users have a better time with our Projects.
  * You can Translate our Project [Stoat Games](https://stoatgames.icu).
  * You can Contribute to the Project [Stoat Games](https://stoatgames.icu) by shooting us emails with Feedback and better code.

# LICENSED UNDER [SML-1.0.0](https://stoatgames.icu/license-repository/licenses/sml-1-0-0/#content)