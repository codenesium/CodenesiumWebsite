+++
title = "Tooling"
description = ""
weight = 1
alwaysopen = true
+++
Codenesium has tooling to help with your development process.

Generated files that are expected to not be modified and generated over are named with a .g.cs extension.

Generated files that are meant to be modified have a .cs extension. 

All generated files have a hash appended to the source code

```
/*<Codenesium>
    <Hash>a3b4672d5db2dbc17932846fa4116b4c</Hash>
</Codenesium>*/
```

This hash is a MD5 hash of the contents of the file. If you modify the file in any way the hash will be invalid. When the generation is ran again we don't overwrite
any files that don't match their hash. This allows you to make changes and be confident Codenesium isn't going to overwrite them.

Files with a .cs extension have the hash and can me modified by you but the hash is there in case you want to prevent modification of those files too. 

Some files like the includes.cs files in each project do not have a hash. These files are not regenerated after the first pass and can me modified. 

In the application menu there is a modified files report. This report will scan your project directory for any files that have a Codenesium hash that doesn't match and returns the output. 

Menu->Modified Files Report

When this report is ran a ModifiedFilesReport.txt file is generated and written to the Codenesium folder in the project. If you want to add exceptions so that modified files you intend to be modified no longer show in this report then you can add entries into the ModifiedSilExceptions.txt file. One entry per line relative to the project root directory.

For a project named TestProject you could add an entry like 
```
TESTPROJECT.API.CLIENT\APICLIENT\ABSTRACTAPICLIENT.G.CS
```
to removed this file from the modified files report. 

The point of this is to be able to see when someone modifies files they shouldn't have in source control and this lets you keep as much of the generation intact as possible so you can regenerate in the future. 