FROM abrarov/msvc-2019
MAINTAINER Jan Kleinert <jan.kleinert@dlr.de>
LABEL Description="MSVC 2019 with Qt5, VTK setup for use with pybind11, python and some dev tools"

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop';"]

ARG qt_version=5.14.1
ARG python_version=3.7.9
ARG vtk_version=9.0.1

COPY requirements.txt requirements.txt

###############################################
# Install software requirements, build tools  #
###############################################

RUN choco install cmake -y; \
    choco install 7zip -y; \
    choco install ninja -y; \
    choco install nsis -y; \
	New-Item -ItemType directory -Path C:\Tools; \
	[Environment]::SetEnvironmentVariable('Path', ($env:Path + ';' + $env:CMAKE_HOME + '\bin;C\Tools\;C:\Program Files\7-Zip\;C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Auxiliary\Build'), [EnvironmentVariableTarget]::Machine);
	
COPY activate_vc2019.ps1 C:/Tools/vcvars_ps.ps1

##################
# Install Python #
##################
# from https://www.gyanblog.com/python/how-install-python-windows-command-line-Dockerfile/ and https://github.com/docker-library/python/blob/master/Dockerfile-windowsservercore.template

# https://github.com/pypa/get-pip
ENV PYTHON_GET_PIP_URL https://github.com/pypa/get-pip/raw/d59197a3c169cef378a22428a3fa99d33e080a5d/get-pip.py
ENV PYTHON_GET_PIP_SHA256 421ac1d44c0cf9730a088e337867d974b91bdce4ea2636099275071878cc189e

RUN \
# unfortunately abrarov/msvc-2019 already has Python installed. Can I use a slimmer base?
    remove-Item –path C:\Python2 -recurse; \
	remove-Item –path C:\Python3 -recurse; \
# from https://www.gyanblog.com/python/how-install-python-windows-command-line-Dockerfile/ and https://github.com/docker-library/python/blob/master/Dockerfile-windowsservercore.template
    $url = ('https://www.python.org/ftp/python/{0}/python-{0}-amd64.exe' -f $env:python_version); \
	Write-Host ('Downloading {0} ...' -f $url); \
	Invoke-WebRequest -Uri $url -OutFile 'python.exe'; \
	\
	Write-Host 'Installing ...'; \
# https://docs.python.org/3/using/windows.html#installing-without-ui
	$exitCode = (Start-Process python.exe -Wait -NoNewWindow -PassThru \
		-ArgumentList @( \
			'/quiet', \
			'InstallAllUsers=1', \
			'PrependPath=1', \
			'Shortcuts=0', \
			'Include_doc=0', \
			'Include_pip=0', \
			'Include_test=0' \
		) \
	).ExitCode; \
	if ($exitCode -ne 0) { \
		Write-Host ('Running python installer failed with exit code: {0}' -f $exitCode); \
		Get-ChildItem $env:TEMP | Sort-Object -Descending -Property LastWriteTime | Select-Object -First 1 | Get-Content; \
		exit $exitCode; \
	} \
	\
# the installer updated PATH, so we should refresh our local value
	$env:PATH = [Environment]::GetEnvironmentVariable('PATH', [EnvironmentVariableTarget]::Machine); \
	\
	Write-Host 'Verifying install ...'; \
	Write-Host '  python --version'; python --version; \
	\
	Write-Host 'Removing ...'; \
	Remove-Item python.exe -Force; \
	Remove-Item $env:TEMP/Python*.log -Force; \
	\
	Write-Host 'Complete.'; \
# install pip
    Write-Host ('Downloading get-pip.py ({0}) ...' -f $env:PYTHON_GET_PIP_URL); \
	[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; \
	Invoke-WebRequest -Uri $env:PYTHON_GET_PIP_URL -OutFile 'get-pip.py'; \
	Write-Host ('Verifying sha256 ({0}) ...' -f $env:PYTHON_GET_PIP_SHA256); \
	if ((Get-FileHash 'get-pip.py' -Algorithm sha256).Hash -ne $env:PYTHON_GET_PIP_SHA256) { \
		Write-Host 'FAILED!'; \
		exit 1; \
	}; \
	\
	Write-Host ('Installing pip ...'); \
	python get-pip.py \
		--disable-pip-version-check \
		--no-cache-dir \
	; \
	Remove-Item get-pip.py -Force; \
	\
	Write-Host 'Verifying pip install ...'; \
	pip --version; \
	\
	Write-Host 'Complete.'; \
	Write-Host ('Installing requirements from requirements.txt') \
	pip install -r requirements.txt

#######################################
# build vtk with VTK_WRAP_PYTHON=ON   #
#######################################

# clone vtk
RUN git clone https://github.com/Kitware/VTK.git; \
    cd VTK; \
    git checkout v$env:vtk_version; \
# configure build with cmake
    New-Item -ItemType directory -Path build; \
	cd build ; \
	. C:\Tools\vcvars_ps.ps1; \
    cmake -G "Ninja" .. \
    -DBUILD_TESTING=OFF \
    -DCMAKE_BUILD_TYPE=Release \
    -DVTK_WRAP_PYTHON=ON \
    -DVTK_PYTHON_VERSION=3 \
    -DPython3_FIND_STRATEGY=LOCATION; \
# build and install vtk
    ninja install; \
# cleanup
    cd ..\..; \
	Remove-Item –path VTK -recurse -force

############################################################################
# build qt from source (offline installer not available for FOSS anymore)  #
############################################################################

# clone qt
RUN git clone https://code.qt.io/qt/qt5.git; \
    cd qt5; \
    git checkout v$env:qt_version; \
    perl init-repository --module-subset=essential,qtscript; \
# configure qt
    New-Item -ItemType directory -Path build; \
	cd build ; \
	. C:\Tools\vcvars_ps.ps1; \
    ..\configure -opensource -nomake examples -nomake tests -confirm-license; \
# build qt
    nmake; nmake install; \
# cleanup
    cd ..\..; \
	Remove-Item –path qt5 -recurse -force
