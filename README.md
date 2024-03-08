# C# Runtime Experiment

## Getting Started

Step 1. run Saffron Runtime simulation (provide gRPC agent calls)

```bash
dotnet run --project example/RuntimeServer
```

Step 2. build your Logic code

```bash
# https://learn.microsoft.com/en-us/dotnet/core/rid-catalog
export DOTNET_RID=linux-arm64

dotnet publish example/Logic \
    /p:NativeLib=Shared \
    --runtime $DOTNET_RID \
    --no-self-contained
```

Step 3. Execute C# Runtime with your Logic

```bash
# https://learn.microsoft.com/en-us/dotnet/core/rid-catalog
export DOTNET_RID=linux-arm64

export LD_LIBRARY_PATH="/workspaces/loc-logic-sdk-cs/example/Logic/bin/Debug/net7.0/$DOTNET_RID/publish"
# export LD_LIBRARY_PATH="/workspaces/loc-logic-sdk-cs/example/Logic/bin/release/net7.0/$DOTNET_RID/publish"
dotnet run --project Runtime -- --runtime-address http://localhost:5224 --execution-id 0 --task-id 0
```

## Pack and Publish SDK

Step 1. package SDK as a .nupkg file

```bash
dotnet pack SDK --configuration Release
```

Step 2.
publish to https://www.nuget.org/

```bash
dotnet nuget push ./SDK/bin/Release/FSTNetwork.LOC.Logic.SDK.0.0.6.nupkg \
    --api-key oy2p5ug6dgdlzno6bxiauz7rxru4qjkezm6nfxvhgl24nu \
    --source https://api.nuget.org/v3/index.json \
    --skip-duplicate
```

or publish to the test server https://int.nugettest.org/

```bash
dotnet nuget push ./SDK/bin/Release/FSTNetwork.LOC.Logic.SDK.0.0.6.nupkg \
    --api-key oy2bnoiytx7r6f3ja2rab6ovhrdc3nhenb3awkmdg6mc7m \
    --source https://apiint.nugettest.org/v3/index.json \
    --skip-duplicate
```

or publish to self-host NuGet server (e.g. BaGet)

```bash
kubectl port-forward svc/baget 8080 --address 192.168.96.80
dotnet nuget push ./SDK/bin/Release/FSTNetwork.LOC.Logic.SDK.0.0.6.nupkg \
    --api-key ERTdMbF49MS6jaaxvXqDntly \
    --source http://192.168.96.80:8080/v3/index.json \
    --skip-duplicate
```

## Download from difference NuGet source

add new NuGet source

```bash
dotnet nuget add source https://apiint.nugettest.org/v3/index.json --name nugettest.org
```

add package from specify source

```bash
dotnet add package FSTNetwork.LOC.Logic.Sdk --version 0.0.6 --source "nugettest.org"
```

download all decencies from specify source

```bash
dotnet restore --source "nugettest.org"
```

## Prepare for an offline environment

```bash
export SDK_VERSION=0.0.17
export DOTNET_RUNTIME_VERSION=7.0.16

# dotnet nuget add source https://apiint.nugettest.org/v3/index.json --name nugettest.org

# clean all cache packages
rm $(find  ~/.nuget/ -name '*.nupkg')

# download necessary packages
dotnet new console -o /tmp/temp-project
dotnet add /tmp/temp-project package FSTNetwork.LOC.Logic.Sdk --version $SDK_VERSION
dotnet add /tmp/temp-project package runtime.linux-x64.Microsoft.DotNet.ILCompiler --version $DOTNET_RUNTIME_VERSION
rm -rf /tmp/temp-project

# upload packages to self-host nuget server
dotnet nuget push ~/.nuget/**/*.nupkg \
    --api-key ERTdMbF49MS6jaaxvXqDntly \
    --source https://baget.cellar.fst.network/v3/index.json \
    --skip-duplicate
```

## Buggy

Sometimes, NuGet cache can prevent you from successfully accessing new versions 🥲

```
$ dotnet add package LOC.Logic.Sdk --version 0.0.3
  Determining projects to restore...
  Writing /tmp/tmpPiPo5u.tmp
info : X.509 certificate chain validation will use the fallback certificate bundle at '/usr/share/dotnet/sdk/7.0.401/trustedroots/codesignctl.pem'.
info : X.509 certificate chain validation will use the fallback certificate bundle at '/usr/share/dotnet/sdk/7.0.401/trustedroots/timestampctl.pem'.
info : Adding PackageReference for package 'LOC.Logic.Sdk' into project '/workspaces/loc-logic-sdk-cs/example/Shared/Shared.csproj'.
info : Restoring packages for /workspaces/loc-logic-sdk-cs/example/Shared/Shared.csproj...
info :   GET https://api.nuget.org/v3-flatcontainer/loc.logic.sdk/index.json
info :   CACHE https://int.nugettest.org/api/v2/FindPackagesById()?id='LOC.Logic.Sdk'&semVerLevel=2.0.0
info :   NotFound https://api.nuget.org/v3-flatcontainer/loc.logic.sdk/index.json 1048ms
error: NU1102: Unable to find package LOC.Logic.Sdk with version (>= 0.0.3)
error:   - Found 1 version(s) in nugettest.org [ Nearest version: 0.0.2 ]
error:   - Found 0 version(s) in nuget.org
error: Package 'LOC.Logic.Sdk' is incompatible with 'all' frameworks in project '/workspaces/loc-logic-sdk-cs/example/Shared/Shared.csproj'.
```

Please clean NuGet cache and try it agent

```
dotnet nuget locals all -c
```
