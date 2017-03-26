### Overview

Typically, pyodbc is installed like any other Python package by running:

`pip install pyodbc`

from a Windows DOS prompt or Unix shell.  See the pip [documentation](https://pip.pypa.io/en/latest/user_guide.html "pip user guide") for more details about the pip utility.

As always when installing modules, you should consider using [Python virtual environments](http://docs.python-guide.org/en/latest/dev/virtualenvs/).

Binary wheels are released for most versions on Windows and macOS.

#### Installing on Linux

When installing pyodbc on Linux, `pip` will download and compile the pyodbc source code. This requires that some related source files from unixODBC be available for the compile to succeed. On Ubuntu systems, all you need to do is run

`sudo apt install unixodbc-dev`

before attempting

`pip install pyodbc`

(Other Linux distributions may require a slightly different command to install the unixODBC development components.)