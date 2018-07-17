# escape=`
FROM microsoft/dotnet-framework:4.7.2-runtime

MAINTAINER dealdiane@netpotential.co.nz

# This DockerFile is based from https://github.com/Microsoft/dotnet-framework-docker/blob/master/4.7.1-windowsservercore-1709/build/Dockerfile

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue'; $VerbosePreference = 'Continue'; "]

ENV NUGET_VERSION=4.4.1 `
    NODE_VERSION=8.10.0 `
    NODE_DOWNLOAD_SHA=936ada36cb6f09a5565571e15eb8006e45c5a513529c19e21d070acf0e50321b `
    VS_BUILDTOOLS_URI=https://download.visualstudio.microsoft.com/download/pr/11835061/e64d79b40219aea618ce2fe10ebd5f0d/vs_BuildTools.exe

# Install NuGet CLI    
RUN New-Item -Type Directory $Env:ProgramFiles\NuGet; `
    Invoke-WebRequest -UseBasicParsing https://dist.nuget.org/win-x86-commandline/v$Env:NUGET_VERSION/nuget.exe -OutFile $Env:ProgramFiles\NuGet\nuget.exe

# Install VS Test Agent
#RUN Invoke-WebRequest -UseBasicParsing https://download.visualstudio.microsoft.com/download/pr/11796489/8a386d27295953ee79281fd1f1832e2d/vs_TestAgent.exe -OutFile vs_TestAgent.exe; `
#    Start-Process vs_TestAgent.exe -ArgumentList '--quiet', '--norestart', '--nocache' -NoNewWindow -Wait; `
#    Remove-Item -Force vs_TestAgent.exe; `
    
# Download log collection utility
RUN $ErrorActionPreference = 'Stop'; `
    $ProgressPreference = 'SilentlyContinue'; `
    $VerbosePreference = 'Continue'; `
    Invoke-WebRequest -Uri https://aka.ms/vscollect.exe -OutFile C:\collect.exe

# Download and install Visual Studio Build Tools
RUN Invoke-WebRequest -Uri $env:VS_BUILDTOOLS_URI -OutFile C:\vs_buildtools.exe; `
    # Installer won't detect DOTNET_SKIP_FIRST_TIME_EXPERIENCE if ENV is used, must use setx /M
    SETX /M DOTNET_SKIP_FIRST_TIME_EXPERIENCE 1; `
    $p = Start-Process -Wait -PassThru -FilePath C:\vs_buildtools.exe -ArgumentList `
      '--add', 'Microsoft.VisualStudio.Component.Roslyn.Compiler', `
      '--add', 'Microsoft.VisualStudio.Workload.WebBuildTools', `
      '--add', 'Microsoft.VisualStudio.Workload.MSBuildTools', `
      '--add', 'Microsoft.VisualStudio.Workload.NetCoreBuildTools', `
      '--add', 'Microsoft.VisualStudio.Component.Node.Tools', `
      '--includeRecommended --quiet --nocache --wait --installPath C:\BuildTools'; `
    if ($ret = $p.ExitCode) { c:\collect.exe; throw ('Install failed with exit code 0x{0:x}' -f $ret) } `
    Remove-Item -Force vs_buildtools.exe; `
    Remove-Item -Force -Recurse \"${Env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\"; `
    Remove-Item -Force -Recurse ${Env:TEMP}\*; `
    Remove-Item -Force -Recurse \"${Env:ProgramData}\Package Cache\"; `
    $env:PATH = 'C:\BuildTools\MSBuild\15.0\Bin\;' + $env:PATH; `
    [Environment]::SetEnvironmentVariable('PATH', $env:PATH, [EnvironmentVariableTarget]::Machine);
 
# Can't get this to work so disabling...    
# ngen assemblies queued by VS installers - must be done in cmd shell to avoid access issues
#SHELL ["cmd", "/S", "/C"]
#RUN set COMPLUS_NGenProtectedProcess_FeatureEnabled=0 `
#    && \Windows\Microsoft.NET\Framework64\v4.0.30319\ngen update
#    # Disabled because we're not installing TestAgent
#    # Workaround VS installer/ngen issue with VSWebLauncher
#    # && \Windows\Microsoft.NET\Framework\v4.0.30319\ngen uninstall "C:\Program Files (x86)\Microsoft Visual Studio\2017\TestAgent\Common7\IDE\VSWebLauncher.exe" `
#    # && \Windows\Microsoft.NET\Framework\v4.0.30319\ngen update
#SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue'; $VerbosePreference = 'Continue'; "]

# Install Targeting Packs
RUN @('4.0', '4.5.2', '4.6.2', '4.7.2') `
    | %{ `
        Invoke-WebRequest -UseBasicParsing https://dotnetbinaries.blob.core.windows.net/referenceassemblies/v${_}.zip -OutFile referenceassemblies.zip; `
        Expand-Archive referenceassemblies.zip -Force -DestinationPath \"${Env:ProgramFiles(x86)}\Reference Assemblies\Microsoft\Framework\.NETFramework\"; `
        Remove-Item -Force referenceassemblies.zip; `
    }


# Install VS Build Tools
#    Invoke-WebRequest -UseBasicParsing https://download.visualstudio.microsoft.com/download/pr/11796503/e64d79b40219aea618ce2fe10ebd5f0d/vs_BuildTools.exe -OutFile vs_BuildTools.exe; `
#    # Installer won't detect DOTNET_SKIP_FIRST_TIME_EXPERIENCE if ENV is used, must use setx /M
#    setx /M DOTNET_SKIP_FIRST_TIME_EXPERIENCE 1; `
#    Start-Process -Wait -PassThru -FilePath vs_BuildTools.exe -ArgumentList `
#      '--add', 'Microsoft.VisualStudio.Component.Roslyn.Compiler', `
#      '--add', 'Microsoft.VisualStudio.Workload.WebBuildTools', `
#      '--add', 'Microsoft.VisualStudio.Workload.MSBuildTools', `
#      '--add', 'Microsoft.VisualStudio.Workload.NetCoreBuildTools', `
#      '--includeRecommended --quiet --nocache --wait --installPath C:\BuildTools'; `
#     '--quiet', '--norestart', '--nocache', '--installPath C:\BuildTools', '--includeRecommended' -NoNewWindow -Wait; `
#   Remove-Item -Force vs_buildtools.exe; `
#   Remove-Item -Force -Recurse \"${Env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\"; `
#   Remove-Item -Force -Recurse ${Env:TEMP}\*; `
#   Remove-Item -Force -Recurse \"${Env:ProgramData}\Package Cache\"
#    $env:PATH = 'C:\BuildTools\MSBuild\15.0\Bin\;' + $env:PATH; `
#    [Environment]::SetEnvironmentVariable('PATH', $env:PATH, [EnvironmentVariableTarget]::Machine);


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

RUN Invoke-Expression ((New-Object Net.Webclient).DownloadString('https://chocolatey.org/install.ps1')) `
    & choco feature disable --name showDownloadProgress