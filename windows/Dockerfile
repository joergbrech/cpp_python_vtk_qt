FROM abrarov/msvc-2019
MAINTAINER Jan Kleinert <jan.kleinert@dlr.de>
LABEL Description="MSVC 2019 with Qt5, VTK setup for use with pybind11, miniforge and some dev tools"

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
	[Environment]::SetEnvironmentVariable('Path', ($env:Path + ';' + $env:CMAKE_HOME + '\bin;C\Tools\;C:\Tools\Miniforge3;C:\Tools\Miniforge3\Scripts;C:\Program Files\7-Zip\;C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Auxiliary\Build'), [EnvironmentVariableTarget]::Machine);
	
COPY activate_vc2019.ps1 C:/Tools/vcvars_ps.ps1

	
###################################################
# Install Miniforge and setup default environment #
###################################################

RUN \
# unfortunately abrarov/msvc-2019 already has Python installed. Can I use a slimmer base?
    remove-Item –path C:\Python2 -recurse; \
	remove-Item –path C:\Python3 -recurse; \
# download Miniforge
    $url = 'https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Windows-x86_64.exe'; \
    Write-Host ('Downloading {0} ...' -f $url); \
    Invoke-WebRequest -Uri $url -OutFile 'Miniforge3-Windows-x86_64.exe'; \
    \
# run the Miniforge installer
    Write-Host 'Installing ...'; \
    $exitCode = (Start-Process 'Miniforge3-Windows-x86_64.exe' -Wait -NoNewWindow -PassThru \
		-ArgumentList @( \
			'/InstallationType=JustMe', \
			'/RegisterPython=0', \
			'/S', \
			'/D=C:\Tools\Miniforge3' \
		)\
	).ExitCode; \
    if ($exitCode -ne 0) { \
		Write-Host ('Running minoforge installer failed with exit code: {0}' -f $exitCode); \
		Get-ChildItem $env:TEMP | Sort-Object -Descending -Property LastWriteTime | Select-Object -First 1 | Get-Content; \
		exit $exitCode; \
	} \
	\
# make sure the PATH is up to date
	$env:PATH = [Environment]::GetEnvironmentVariable('PATH', [EnvironmentVariableTarget]::Machine); \
    Remove-Item –path 'Miniforge3-Windows-x86_64.exe' -Force; \
    conda init; \
# reload the Powershell profile modified by conda init
	. C:\Users\ContainerAdministrator\Documents\WindowsPowerShell\profile.ps1; \
    conda update conda; \
# install python and some dev stuff into default environment
    conda install python=$env:python_version pip

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
	
#############################
# Install some python tools #
#############################
# This is the last step, because pip will download PyQt5 source, which requires a Qt installation.
# So doing this as part of the Python install will error

RUN pip install -r requirements.txt
