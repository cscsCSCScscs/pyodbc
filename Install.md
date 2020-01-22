### Overview

Typically, pyodbc is installed like any other Python package by running:

~~~
pip install pyodbc
~~~

from a Windows DOS prompt or Unix shell.  See the pip [documentation](https://pip.pypa.io/en/latest/user_guide.html "pip user guide") for more details about the pip utility.

As always when installing modules, you should consider using [Python virtual environments](http://docs.python-guide.org/en/latest/dev/virtualenvs/).

Binary wheels are released for most versions on Windows and macOS.

On Windows, you should install [Microsoft Visual C++ Redistributable for Visual Studio 2015, 2017 and 2019](https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads) when using the precompiled binaries from pip.

### Installing on Linux

When installing pyodbc on Linux, `pip` will download and compile the pyodbc source code. This requires that some related components and source files be available for the compile to succeed. 

#### Ubuntu 18.04

On Ubuntu systems, all you need to do is run

~~~
sudo apt install python3-pip
sudo apt install unixodbc-dev
pip3 install --user pyodbc
~~~

#### Debian Stretch

Similar to Ubuntu, you need to install `unixodbc-dev`, but you will also need to install `gcc` and `g++`. Note `gcc` package is automatically installed when installing `g++`

~~~
apt-get update
apt-get install g++
apt-get install unixodbc-dev
pip install pyodbc
~~~

#### CentOS 7

From a clean minimal install of CentOS 7, the following steps were required:

~~~
sudo yum install epel-release
sudo yum install python-pip gcc-c++ python-devel unixODBC-devel
pip install --user pyodbc
~~~

#### Fedora 27

~~~
sudo dnf install redhat-rpm-config gcc-c++ python3-devel unixODBC-devel
pip3 install --user pyodbc
~~~

#### OpenSUSE

Similar to Fedora, the following packages were required after a clean install of OpenSUSE Leap 42.3

~~~
sudo zypper install gcc-c++ python3-devel unixODBC-devel
pip3 install --user pyodbc
~~~
