# Testing & Validation Guide

This document provides step-by-step instructions to test and validate the .NET standards.

## Prerequisites

- .NET SDK 8.0+ installed (`dotnet --version`)
- Git installed
- Text editor or IDE (VS Code, Rider, Visual Studio)

## Test 1: Isolated Project Test

This test creates a new project from scratch and applies the standards.

### Steps

1. **Create test directory:**
   ```bash
   mkdir test-isolated
   cd test-isolated
   ```

2. **Copy standard files:**
   ```bash
   cp /path/to/dotnet-standards/.editorconfig .
   cp /path/to/dotnet-standards/Directory.Build.props .
   cp /path/to/dotnet-standards/Directory.Build.targets .
   cp /path/to/dotnet-standards/global.json .
   ```

3. **Create a simple class library:**
   ```bash
   dotnet new classlib -n TestLib
   cd TestLib
   ```

4. **Create a sample class with intentional style violations:**
   Create `BadStyle.cs`:
   ```csharp
   namespace TestLib {  // Should trigger file-scoped namespace warning
       public class badClass {  // Should trigger naming convention warning
           public string Name;  // Should trigger accessibility modifier warning
           
           public string GetName() {
               return this.Name;  // Should trigger 'this.' qualification warning
           }
       }
   }
   ```

5. **Build and check for warnings:**
   ```bash
   cd ..
   dotnet build TestLib
   ```

6. **Expected output:**
   - ✅ Build should succeed
   - ✅ Should show MinVer version message (if Git initialized)
   - ✅ Should show style warnings for:
     - File-scoped namespace
     - Type naming convention
     - Accessibility modifiers
     - `this.` qualification

7. **Fix the style violations:**
   Update `BadStyle.cs`:
   ```csharp
   namespace TestLib;

   public class GoodClass
   {
       private string _name;
       
       public string GetName()
       {
           return _name;
       }
   }
   ```

8. **Rebuild:**
   ```bash
   dotnet build TestLib
   ```

9. **Expected output:**
   - ✅ Build succeeds with no warnings (or fewer warnings)

### Validation Checklist

- [ ] Project builds successfully
- [ ] Directory.Build.props is detected (check build output)
- [ ] Directory.Build.targets is detected (MinVer message shown)
- [ ] .editorconfig rules are enforced (warnings for violations)
- [ ] global.json SDK version is respected

## Test 2: NuGet Package Build

This test validates that the NuGet package builds correctly.

### Steps

1. **Navigate to standards repo:**
   ```bash
   cd /path/to/dotnet-standards
   ```

2. **Build the package:**
   ```bash
   dotnet pack Atypical.Build.Standards.csproj -c Release -o ./artifacts
   ```

3. **Verify package contents:**
   ```bash
   # Extract and inspect the .nupkg
   mkdir temp-extract
   cd temp-extract
   unzip ../artifacts/Atypical.Build.Standards.1.0.0.nupkg
   ```

4. **Expected contents:**
   ```
   content/
     .editorconfig
     Directory.Build.props
     Directory.Build.targets
     global.json
   contentFiles/
     any/
       any/
         .editorconfig
         Directory.Build.props
         Directory.Build.targets
         global.json
   README.md
   LICENSE
   Atypical.Build.Standards.nuspec
   ```

5. **Check .nuspec metadata:**
   ```bash
   cat Atypical.Build.Standards.nuspec
   ```

6. **Expected metadata:**
   - Package ID: `Atypical.Build.Standards`
   - Version: `1.0.0`
   - Authors: `Philippe Matray`
   - License: `MIT`
   - Description present
   - Tags present

### Validation Checklist

- [ ] Package builds without errors
- [ ] Package contains all required files
- [ ] Package metadata is correct
- [ ] README.md included
- [ ] LICENSE included

## Test 3: Package Consumption Test

This test installs the package in a new project.

### Steps

1. **Create local NuGet source:**
   ```bash
   mkdir ~/local-nuget
   cp /path/to/dotnet-standards/artifacts/*.nupkg ~/local-nuget/
   ```

2. **Add local source:**
   ```bash
   dotnet nuget add source ~/local-nuget --name Local
   ```

3. **Create test project:**
   ```bash
   mkdir test-consumption
   cd test-consumption
   dotnet new classlib -n ConsumerLib
   cd ConsumerLib
   ```

4. **Install the package:**
   ```bash
   dotnet add package Atypical.Build.Standards --version 1.0.0 --source Local
   ```

5. **Create a sample class:**
   ```csharp
   namespace ConsumerLib;

   public class SampleClass
   {
       private readonly string _value;

       public SampleClass(string value)
       {
           _value = value;
       }

       public string GetValue() => _value;
   }
   ```

6. **Build:**
   ```bash
   dotnet build
   ```

7. **Expected output:**
   - ✅ Build succeeds
   - ✅ Standards are applied (check for MinVer message)
   - ✅ No style warnings (code follows conventions)

### Validation Checklist

- [ ] Package installs successfully
- [ ] Standards files are available in project
- [ ] Build uses standards (MinVer, Source Link, etc.)
- [ ] Code style is enforced

## Test 4: MinVer Versioning Test

This test validates MinVer integration.

### Steps

1. **Create test project with Git:**
   ```bash
   mkdir test-minver
   cd test-minver
   git init
   cp /path/to/dotnet-standards/.editorconfig .
   cp /path/to/dotnet-standards/Directory.Build.props .
   cp /path/to/dotnet-standards/Directory.Build.targets .
   dotnet new classlib -n VersionTest
   ```

2. **Initial build (no tags):**
   ```bash
   dotnet build VersionTest
   ```
   
   Expected: MinVer assigns `0.0.0-alpha.0` or similar default version.

3. **Create first tag:**
   ```bash
   git add .
   git commit -m "Initial commit"
   git tag v0.1.0
   ```

4. **Build again:**
   ```bash
   dotnet build VersionTest
   ```
   
   Expected: MinVer uses `0.1.0`.

5. **Make changes and build (dirty):**
   ```bash
   echo "// Change" >> VersionTest/Class1.cs
   dotnet build VersionTest
   ```
   
   Expected: MinVer appends height/commit hash (e.g., `0.1.0-alpha.0.1+abcd123`).

6. **Commit and build:**
   ```bash
   git add .
   git commit -m "Change"
   dotnet build VersionTest
   ```
   
   Expected: Clean version with height (e.g., `0.1.1-alpha.0`).

### Validation Checklist

- [ ] MinVer works without tags (default version)
- [ ] MinVer detects Git tags correctly
- [ ] MinVer appends commit info for dirty builds
- [ ] MinVer message appears in build output

## Test 5: Source Link Test

This test validates Source Link integration.

### Steps

1. **Create test project on GitHub:**
   ```bash
   mkdir test-sourcelink
   cd test-sourcelink
   git init
   git remote add origin https://github.com/phmatray/test-sourcelink.git
   ```

2. **Copy standards and create project:**
   ```bash
   cp /path/to/dotnet-standards/.editorconfig .
   cp /path/to/dotnet-standards/Directory.Build.props .
   cp /path/to/dotnet-standards/Directory.Build.targets .
   dotnet new classlib -n SourceLinkTest
   ```

3. **Build and pack:**
   ```bash
   git add .
   git commit -m "Initial commit"
   git tag v1.0.0
   dotnet pack SourceLinkTest -c Release
   ```

4. **Inspect symbols:**
   ```bash
   # Check that .snupkg is created
   ls SourceLinkTest/bin/Release/
   ```

5. **Expected output:**
   - ✅ `SourceLinkTest.1.0.0.nupkg`
   - ✅ `SourceLinkTest.1.0.0.snupkg` (symbol package)

### Validation Checklist

- [ ] Symbol package (.snupkg) is generated
- [ ] Source Link metadata is embedded
- [ ] Repository URL is correct

## Test 6: CI Build Test

This test validates deterministic builds in CI.

### Steps

1. **Simulate CI environment:**
   ```bash
   export GITHUB_ACTIONS=true
   ```

2. **Build:**
   ```bash
   dotnet build VersionTest
   ```

3. **Check for ContinuousIntegrationBuild:**
   ```bash
   dotnet build VersionTest -v detailed | grep ContinuousIntegrationBuild
   ```

4. **Expected output:**
   - ✅ `ContinuousIntegrationBuild=true` appears in build log

5. **Clean up:**
   ```bash
   unset GITHUB_ACTIONS
   ```

### Validation Checklist

- [ ] CI flag is detected
- [ ] Deterministic build is enabled
- [ ] No warnings related to CI builds

## Troubleshooting

### Issue: MinVer not showing version

**Solution:**
- Ensure Git is initialized: `git init`
- Create at least one tag: `git tag v0.1.0`
- MinVer is skipped in Debug mode by default (build in Release)

### Issue: .editorconfig not enforced

**Solution:**
- Ensure `root = true` is set
- Restart your IDE
- Check that `EnforceCodeStyleInBuild=true` in Directory.Build.props

### Issue: Package not found when installing

**Solution:**
- Verify local NuGet source: `dotnet nuget list source`
- Clear NuGet cache: `dotnet nuget locals all --clear`
- Rebuild package: `dotnet pack -c Release`

### Issue: "SDK version not found"

**Solution:**
- Update global.json to match your installed SDK
- Or remove global.json to use default SDK

## Success Criteria

All tests pass when:

- ✅ Isolated project builds with standards
- ✅ NuGet package builds successfully
- ✅ Package can be consumed by other projects
- ✅ MinVer versioning works correctly
- ✅ Source Link generates symbol packages
- ✅ CI detection works (deterministic builds)

---

**Next Step:** Run these tests on a machine with .NET SDK installed, then proceed to pilot repo testing (VatBe).
