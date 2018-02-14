### Overview

Typically, pyodbc is installed like any other Python package by running:

~~~
pip install pyodbc
~~~

from a Windows DOS prompt or Unix shell.  See the pip [documentation](https://pip.pypa.io/en/latest/user_guide.html "pip user guide") for more details about the pip utility.

As always when installing modules, you should consider using [Python virtual environments](http://docs.python-guide.org/en/latest/dev/virtualenvs/).

Binary wheels are released for most versions on Windows and macOS.

### Installing on Linux

When installing pyodbc on Linux, `pip` will download and compile the pyodbc source code. This requires that some related components and source files be available for the compile to succeed. 

#### Ubuntu 16.04

On Ubuntu systems, all you need to do is run

~~~
sudo apt install python3-pip  # or `sudo apt install python-pip` for Python 2.x
sudo apt install unixodbc-dev
sudo pip3 install pyodbc  # or `sudo pip install pyodbc` for Python 2.x
~~~

#### CentOS 7

From a clean minimal install of CentOS 7, the following steps were required:

~~~
sudo yum install epel-release
sudo yum install python-pip
sudo yum install gcc-c++
sudo yum install python-devel
sudo yum install unixODBC-devel
sudo pip install pyodbc
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