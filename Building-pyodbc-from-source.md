### Overview

#### Obtaining Source
The source for released versions are provided in zip files on the front page of this repository.

If you are going to work on pyodbc (changes are welcome!), best fork the repository from github so I can easily pull changes from your fork.

If you want an unreleased version from github but don't have git installed, choose the branch and commit you want and use the download button. It will offer you a tar or zip file.

#### Compiling
pyodbc is built using [distutils](http://docs.python.org/library/distutils.html), which comes with Python. If you have the appropriate C++ compiler installed (see below), run the following in the pyodbc source directory:

`python setup.py build`

To install after a successful build:

`python setup.py install`

#### Version
pyodbc uses git tags in the format _major.minor.patch_ for versioning. If the commit being built has a tag, such as "3.0.7", then that is used for the version. If the commit is not tagged, then it is assumed that the commit is a beta for an upcoming release. The patch number is incremented, such as "3.0.7", and the number of commits since the previous release is appended: 3.0.7-beta3.

If you are building from inside a git repository, the build script will automatically determine the correct version from the tags (using git describe).

If you downloaded the source as a zip from the front page, the version will be in the text file PGK-INFO. The build script will read the version from that text file.

If the version cannot be determined for some reason, you will see a warning about this and the script will build version 2.1.0. Do not distribute this version since you won't be able to track its actual version!

### Operating Systems

#### Windows
On Windows, you must use the appropriate Microsoft Visual C++ compiler for the version of pyodbc you wish to compile.  This is a Python requirement, not a pyodbc one. This page on the Python wiki provides information about compiling Python code on Windows: https://wiki.python.org/moin/WindowsCompilers

To build Python 2.4 or 2.5 versions, you will need the Visual Studio 2003 .NET compiler. Unfortunately there is no free version of this.

For Python 2.6, 2.7, 3.0, 3.1 and 3.2, use the Visual C++ 2008 compiler. There is a free version of this, Visual C++ 2008 Express.

For Python 3.3 and 3.4, use the Visual C++ 2010 compiler. There is a free version of this, Visual C++ 2010 Express, although that version does not inherently allow 64-bit builds to be compiled. To compile 64-bit builds with Visual C++ 2010 Express, follow these instructions: http://blog.ionelmc.ro/2014/12/21/compiling-python-extensions-on-windows/

Once the build has been compiled, you can create a Windows installer executable by running:

`python setup.py bdist_wininst`

#### Other
To build on other operating systems, use the gcc compiler.

On Linux, pyodbc is typically built using the unixODBC headers, so you will need unixODBC and its headers installed. On a RedHat/CentOS/Fedora box, this means you would need to install unixODBC-devel:

`yum install unixODBC-devel`