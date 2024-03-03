# Stage 1: Download Python Installer and get-pip.py Script
FROM mcr.microsoft.com/windows/servercore:ltsc2022 AS downloader
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Set the environment variables for the Python version and get-pip.py script location and checksum
ENV PYTHON_VERSION 3.11.8
ENV PYTHON_GET_PIP_URL https://github.com/pypa/get-pip/raw/dbf0c85f76fb6e1ab42aa672ffca6f0a675d9ee4/public/get-pip.py
ENV PYTHON_GET_PIP_SHA256 dfe9fd5c28dc98b5ac17979a953ea550cec37ae1b47a5116007395bfacff2ab9

# Download Python installer
RUN $url = ('https://www.python.org/ftp/python/{0}/python-{1}-amd64.exe' -f ($env:PYTHON_VERSION -replace '[a-z]+[0-9]*$', ''), $env:PYTHON_VERSION); \
    Write-Host ('Downloading Python {0} ...' -f $url); \
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; \
    Invoke-WebRequest -Uri $url -OutFile 'python.exe';

# Download get-pip.py script
RUN Write-Host ('Downloading get-pip.py ...'); \
    Invoke-WebRequest -Uri $env:PYTHON_GET_PIP_URL -OutFile 'get-pip.py'; \
    Write-Host ('Verifying sha256 ...'); \
    if ((Get-FileHash 'get-pip.py' -Algorithm sha256).Hash -ne $env:PYTHON_GET_PIP_SHA256) { \
        Write-Host 'FAILED!'; \
        exit 1; \
    };

# Stage 2: Build the final image, install Python and pip
FROM mcr.microsoft.com/windows/nanoserver:ltsc2022
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

COPY --from=downloader python.exe python.exe
COPY --from=downloader get-pip.py get-pip.py

# Install Python without UI and without bundled pip
RUN Write-Host 'Installing Python ...'; \
    $exitCode = (Start-Process python.exe -Wait -NoNewWindow -PassThru \
        -ArgumentList @( \
            '/quiet', \
            'InstallAllUsers=1', \
            'TargetDir=C:\Python', \
            'PrependPath=1', \
            'Shortcuts=0', \
            'Include_doc=0', \
            'Include_pip=0', \
            'Include_test=0' \
        ) \
    ).ExitCode; \
    if ($exitCode -ne 0) { \
        Write-Host ('Python installer failed with exit code: {0}' -f $exitCode); \
        exit $exitCode; \
    }; \
    Remove-Item python.exe -Force;

# Install pip from the downloaded script
RUN Write-Host 'Installing pip ...'; \
    python get-pip.py --no-cache-dir --no-compile; \
    Remove-Item get-pip.py -Force;

# Verify pip installation
RUN pip --version;

# Optionally, you can install setuptools here as well, but this step is generally handled by the get-pip.py script
# RUN pip install setuptools

# Set the default command for the container
CMD ["python"]