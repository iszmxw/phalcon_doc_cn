安装
============
PHP框架的安装和普通php编写的框架或者是库有些不同，我们可以直接下载编译好的扩展或者从源码编译安装。

PHP extensions require a slightly different installation method to a traditional php-based library or framework.
You can either download a binary package for the system of your choice or build it from the sources.

Windows
-------
在Windows安装Phalcon 我们需要下载 phalcon dll库,然后编辑php.ini 在最后添加以下代码

To use phalcon on Windows you can download_ a DLL library. Edit your php.ini file and then append at the end:

.. code-block:: bash

    extension=php_phalcon.dll

重启web服务

Restart your webserver.

下面的视频演示了在windows平台如何一步步安装phalcon （国外的视频，需要翻墙）

The following screencast is a step-by-step guide to install Phalcon on Windows:

.. raw:: html

    <div align="center"><iframe src="http://player.vimeo.com/video/40265988" width="500" height="266" frameborder="0" webkitAllowFullScreen mozallowfullscreen allowFullScreen></iframe></div>

如果不知道该下载哪个DLL，可以参考下面的教程。

If you don't know what DLL to download, use the following script to figure it out.

相关教程
^^^^^^^^^^^^^^
.. toctree::
    :maxdepth: 1

    xampp
    wamp

Linux/Solaris
-------------
在Linux/Solaris平台可以通过下载编译源码轻松安装。

On a Linux/Solaris system you can easily compile and install the extension from the source code:

系统要求
^^^^^^^^^^^^
基础要求环境

Prerequisite packages are:

* PHP >= 5.3 development resources
* GCC compiler (Linux/Solaris)
* Git (if not already installed in your system - unless you download the package from GitHub and upload it on your server via FTP/SFTP)

其他平台需要的特定包

Specific packages for common platforms:

.. code-block:: bash

    #Ubuntu
    sudo apt-get install php5-dev libpcre3-dev gcc make php5-mysql

    # Suse
    sudo yast -i gcc make autoconf2.13 php5-devel php5-pear php5-mysql

    # CentOS/RedHat/Fedora
    sudo yum install php-devel pcre-devel gcc make

    # Solaris
    pkg install gcc-45 php-53 apache-php53

编译
^^^^^^^^^^^
创建扩展

Creating the extension:

.. code-block:: bash

    git clone --depth=1 git://github.com/phalcon/cphalcon.git
    cd cphalcon/build
    sudo ./install

添加扩展到php配置	
	
Add extension to your php configuration:

.. code-block:: bash

    # Suse: Add this line in your php.ini
    extension=phalcon.so

    # Centos/RedHat/Fedora: Add a file called phalcon.ini in /etc/php.d/ with this content:
    extension=phalcon.so

    # Ubuntu/Debian: Add a file called 30-phalcon.ini in /etc/php5/conf.d/ with this content:
    extension=phalcon.so

    # Debian with php5-fpm: Add a file called 30-phalcon.ini in /etc/php5/fpm/conf.d/30-phalcon.ini with this content:
    extension=phalcon.so

重启web服务	
	
Restart the webserver.

如果运行debian php5-fpm 重启下它

If you are running Debian with php5-fpm, restart it:

.. code-block:: bash

    sudo service php5-fpm restart

phalcon可以自动识别所在的平台，我们可以强制为其他平台编译。	
	
Phalcon automatically detects your architecture, however, you can force the compilation for a specific architecture:

.. code-block:: bash

    cd cphalon/build
    sudo ./install 32bits
    sudo ./install 64bits
    sudo ./install safe

如果自动安装失败了，可以尝试手动编辑扩展。	
	
If the automatic installer fails try building the extension manually:

.. code-block:: bash

    cd cphalcon/build/64bits
    export CFLAGS="-O2 --fvisibility=hidden"
    ./configure --enable-phalcon
    make && sudo make install

Mac OS X
--------


On a Mac OS X system you can compile and install the extension from the source code:

系统要求
^^^^^^^^^^^^
Prerequisite packages are:

* PHP >= 5.4 development resources
* XCode

.. code-block:: bash

    # brew
    brew tap homebrew/homebrew-php
    brew install php54-phalcon
    brew install php55-phalcon
    brew install php56-phalcon

    # MacPorts
    sudo port install php54-phalcon
    sudo port install php55-phalcon
    sudo port install php56-phalcon

添加扩展到php配置
	
Add extension to your php configuration:


FreeBSD
-------
A port is available for FreeBSD. Just only need these simple line commands to install it:

.. code-block:: bash

    pkg_add -r phalcon

or

.. code-block:: bash

    export CFLAGS="-O2 --fvisibility=hidden"
    cd /usr/ports/www/phalcon && make install clean

Installation Notes
------------------
Installation notes for Web Servers:

.. toctree::
    :maxdepth: 1

    apache
    nginx
    cherokee
    built-in

.. _download : http://phalconphp.com/en/download
