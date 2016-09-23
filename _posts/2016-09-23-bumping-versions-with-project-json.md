---
title: Bumping versions with project.json libraries
---

We follow [semantic versioning](http://semver.org/) rules for our libraries and most of them 
already use the new `project.json` project format.

project.json files allow you to use a `*` for the "pre-release" part of your version (e.g. `1.0.0-*`). 
The `dotnet cli` will then replace the `*` with a defined value (e.g. a CI server build number) 
when you add the `--version-suffix` parameter:
```
// this will result in version "1.0.0-rc1"
dotnet pack -c Release --version-suffix "rc1"
```

Unfortunately, it's not possible to use the `*` for any other part of the version number. This means, whenever you want to
update your library version, you manually have to update the `version` and `dependencies` parts of your libraries.

To make this easier, we created a [bump-version.ps1](https://gist.github.com/cwe1ss/5641f809a658ca5923e3fd929504fd07) 
powershell script that automatically updates these parts.

To explain how it works, let's assume we have a solution with the following projects and dependencies:
```
bump-version.ps1
src\MyLibrary\project.json
 - version: 1.2.3-*
 - dependencies: none
src\MyLibrary.Addon\project.json
 - version: 1.2.3-*
 - dependencies:
   - MyLibrary: 1.2.3-*
test\MyLibrary.Tests\project.json
   - dependencies:
     - MyLibrary: 1.2.3-*
test\MyLibrary.Addon.Tests\project.json
   - dependencies:
     - MyLibrary.Addon: 1.2.3-*
```

The following variables must be set in `bump-version.ps1` in order for it to work properly:
```powershell
# The current version will be read from this file
$projectFile = "src\MyLibrary\project.json"

# All versions and dependencies starting with this name will be updated.
$packagePrefix = "MyLibrary"
```
That's it. The script can now be called with one of the following parameters and it will automatically update 
all libraries in the solution.
```powershell
.\bump-version.ps1 -Patch            # results in 1.2.4-*
.\bump-version.ps1 -Minor            # results in 1.3.0-*
.\bump-version.ps1 -Major            # results in 2.0.0-*
.\bump-version.ps1 -Version "2.3.4"  # results in 2.3.4-*
```

Here's the source code of the script. If you have any questions or feedback, please post it as a comment to the gist!

{% gist cwe1ss/5641f809a658ca5923e3fd929504fd07 %}
