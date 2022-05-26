# escape=`
FROM mcr.microsoft.com/dotnet/framework/sdk:4.8

MAINTAINER dealdiane@netpotential.co.nz

# This DockerFile was based from 
#   https://github.com/microsoft/dotnet-framework-docker/blob/main/src/sdk/4.8/windowsservercore-ltsc2019/Dockerfile

ENV NUGET_VERSION=6.1.0 `
    NODE_VERSION=15.10.0 `
    NODE_DOWNLOAD_SHA=2BF72DA4E5DDC485599F2ECA54AB7C59001D70BAEC7FF2BB050D9F4ED1B066A6 `
    VS_BUILDTOOLS_URI=https://aka.ms/vs/17/release/vs_buildtools.exe `
    VS_TESTAGENT_URI=https://download.visualstudio.microsoft.com/download/pr/dc2793e9-7b80-4f11-9e33-85833e8921a6/129ac716764b517709d969cad9e89c4472451c0c7c746f4f1e3141ddcef885c8/vs_TestAgent.exe `
    ROSLYN_COMPILER_LOCATION="C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\MSBuild\Current\Bin\Roslyn" `
    DOTNET_GENERATE_ASPNET_CERTIFICATE=false

# Install NuGet CLI
RUN mkdir "%ProgramFiles%\NuGet\latest" `
    && curl -fSLo "%ProgramFiles%\NuGet\nuget.exe" https://dist.nuget.org/win-x86-commandline/v%NUGET_VERSION%/nuget.exe `
    && mklink "%ProgramFiles%\NuGet\latest\nuget.exe" "%ProgramFiles%\NuGet\nuget.exe"

# Install VS components
RUN `
    # Install VS Test Agent
    curl -fSLo vs_TestAgent.exe %VS_TESTAGENT_URI% `
    && start /w vs_TestAgent --quiet --norestart --nocache --wait `
    && powershell -Command "if ($err = dir $Env:TEMP -Filter dd_setup_*_errors.log | where Length -gt 0 | Get-Content) { throw $err }" `
    && del vs_TestAgent.exe `
    `
    # Install VS Build Tools
    && curl -fSLo vs_BuildTools.exe %VS_BUILDTOOLS_URI% `
    && start /w vs_BuildTools ^ `
        --add Microsoft.Component.ClickOnce.MSBuild ^ `
        --add Microsoft.Net.Component.4.8.SDK ^ `
        --add Microsoft.NetCore.Component.Runtime.3.1 ^ `
        --add Microsoft.NetCore.Component.Runtime.5.0 ^ `
        --add Microsoft.NetCore.Component.Runtime.6.0 ^ `
        --add Microsoft.NetCore.Component.SDK ^ `
        --add Microsoft.VisualStudio.Component.NuGet.BuildTools ^ `
        --add Microsoft.VisualStudio.Component.WebDeploy ^ `
        --add Microsoft.VisualStudio.Web.BuildTools.ComponentGroup ^ `
        --add Microsoft.VisualStudio.Workload.MSBuildTools ^ `
        --add Microsoft.VisualStudio.Workload.VisualStudioExtensionBuildTools ^ `
        --quiet --norestart --nocache --wait `
    && powershell -Command "if ($err = dir $Env:TEMP -Filter dd_setup_*_errors.log | where Length -gt 0 | Get-Content) { throw $err }" `
    && del vs_BuildTools.exe `
    `
    # Trigger dotnet first run experience by running arbitrary cmd
    && "%ProgramFiles%\dotnet\dotnet" help `
    `
    # Workaround for issues with 32-bit ngen
    && %windir%\Microsoft.NET\Framework\v4.0.30319\ngen uninstall "%ProgramFiles(x86)%\Microsoft Visual Studio\2022\TestAgent\Common7\IDE\Extensions\TestPlatform\testhost.arm64.exe" `
    && %windir%\Microsoft.NET\Framework\v4.0.30319\ngen uninstall "%ProgramFiles(x86)%\Microsoft Visual Studio\2022\TestAgent\Common7\IDE\Extensions\TestPlatform\testhost.net452.arm64.exe" `
    && %windir%\Microsoft.NET\Framework\v4.0.30319\ngen uninstall "%ProgramFiles(x86)%\Microsoft Visual Studio\2022\TestAgent\Common7\IDE\Extensions\TestPlatform\testhost.net46.arm64.exe" `
    && %windir%\Microsoft.NET\Framework\v4.0.30319\ngen uninstall "%ProgramFiles(x86)%\Microsoft Visual Studio\2022\TestAgent\Common7\IDE\Extensions\TestPlatform\testhost.net461.arm64.exe" `
    && %windir%\Microsoft.NET\Framework\v4.0.30319\ngen uninstall "%ProgramFiles(x86)%\Microsoft Visual Studio\2022\TestAgent\Common7\IDE\Extensions\TestPlatform\testhost.net462.arm64.exe" `
    && %windir%\Microsoft.NET\Framework\v4.0.30319\ngen uninstall "%ProgramFiles(x86)%\Microsoft Visual Studio\2022\TestAgent\Common7\IDE\Extensions\TestPlatform\testhost.net47.arm64.exe" `
    && %windir%\Microsoft.NET\Framework\v4.0.30319\ngen uninstall "%ProgramFiles(x86)%\Microsoft Visual Studio\2022\TestAgent\Common7\IDE\Extensions\TestPlatform\testhost.net471.arm64.exe" `
    && %windir%\Microsoft.NET\Framework\v4.0.30319\ngen uninstall "%ProgramFiles(x86)%\Microsoft Visual Studio\2022\TestAgent\Common7\IDE\Extensions\TestPlatform\testhost.net472.arm64.exe" `
    && %windir%\Microsoft.NET\Framework\v4.0.30319\ngen uninstall "%ProgramFiles(x86)%\Microsoft Visual Studio\2022\TestAgent\Common7\IDE\Extensions\TestPlatform\testhost.net48.arm64.exe" `
    `
    # Workaround for issues with 64-bit ngen
    && %windir%\Microsoft.NET\Framework64\v4.0.30319\ngen uninstall "%ProgramFiles(x86)%\Microsoft SDKs\Windows\v10.0A\bin\NETFX 4.8 Tools\SecAnnotate.exe" `
    && %windir%\Microsoft.NET\Framework64\v4.0.30319\ngen uninstall "%ProgramFiles(x86)%\Microsoft SDKs\Windows\v10.0A\bin\NETFX 4.8 Tools\WinMDExp.exe" `
    && %windir%\Microsoft.NET\Framework64\v4.0.30319\ngen uninstall "%ProgramFiles(x86)%\Microsoft Visual Studio\2022\BuildTools\MSBuild\Current\Bin\arm64\MSBuild.exe" `
    `
    # ngen assemblies queued by VS installers
    && %windir%\Microsoft.NET\Framework64\v4.0.30319\ngen update `
    && %windir%\Microsoft.NET\Framework\v4.0.30319\ngen update `
    `
    # Cleanup
    && (for /D %i in ("%ProgramFiles(x86)%\Microsoft Visual Studio\Installer\*") do rmdir /S /Q "%i") `
    && (for %i in ("%ProgramFiles(x86)%\Microsoft Visual Studio\Installer\*") do if not "%~nxi" == "vswhere.exe" del "%~i") `
    && powershell Remove-Item -Force -Recurse "%TEMP%\*" `
    && rmdir /S /Q "%ProgramData%\Package Cache"

# Set PATH in one layer to keep image size down.
RUN powershell setx /M PATH $(${Env:PATH} `
    + \";${Env:ProgramFiles}\NuGet\" `
    + \";${Env:ProgramFiles(x86)}\Microsoft Visual Studio\2022\TestAgent\Common7\IDE\CommonExtensions\Microsoft\TestWindow\" `
    + \";${Env:ProgramFiles(x86)}\Microsoft Visual Studio\2022\BuildTools\MSBuild\Current\Bin\" `
    + \";${Env:ProgramFiles(x86)}\Microsoft SDKs\Windows\v10.0A\bin\NETFX 4.8 Tools\" `
    + \";${Env:ProgramFiles(x86)}\Microsoft SDKs\ClickOnce\SignTool\")

# Install Targeting Packs
RUN powershell " `
    $ErrorActionPreference = 'Stop'; `
    $ProgressPreference = 'SilentlyContinue'; `
    @('4.0', '4.5.2', '4.6.2', '4.7.2', '4.8') `
    | %{ `
        Invoke-WebRequest `
            -UseBasicParsing `
            -Uri https://dotnetbinaries.blob.core.windows.net/referenceassemblies/v${_}.zip `
            -OutFile referenceassemblies.zip; `
        Expand-Archive referenceassemblies.zip -DestinationPath \"${Env:ProgramFiles(x86)}\Reference Assemblies\Microsoft\Framework\.NETFramework\"; `
        Remove-Item -Force referenceassemblies.zip; `
    }"

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Install node
# From: https://github.com/aspnet/aspnet-docker/blob/master/1.1/nanoserver/sdk/Dockerfile
RUN Invoke-WebRequest https://nodejs.org/dist/v${env:NODE_VERSION}/node-v${env:NODE_VERSION}-win-x64.zip -outfile node.zip; `
    if ((Get-FileHash node.zip -Algorithm sha256).Hash -ne $env:NODE_DOWNLOAD_SHA) { `
        Write-Host 'NODEJS CHECKSUM VERIFICATION FAILED!'; `
        exit 1; `
    }; `
    Expand-Archive node.zip -DestinationPath "C:/nodejs-tmp/"; `
    Move-Item "C:/nodejs-tmp/node-v${env:NODE_VERSION}-win-x64" -Destination "C:\nodejs"; `
    Remove-Item -Force "C:/nodejs-tmp/"; `
    Remove-Item -Force node.zip; `
    $env:PATH = 'C:\nodejs\;' + $env:PATH; `
    [Environment]::SetEnvironmentVariable('PATH', $env:PATH, [EnvironmentVariableTarget]::Machine);
    
# Install chocolatey    
ENV chocolateyUseWindowsCompression false

RUN Invoke-Expression ((New-Object Net.Webclient).DownloadString('https://chocolatey.org/install.ps1')); `
    &choco feature disable --name showDownloadProgress