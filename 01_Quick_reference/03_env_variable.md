# perl @INC


```
$ osc build
Can't locate Build.pm in @INC ...
$ perl -V
$ export PERL5LIB=/usr/lib/obs-build
```

# LD_LIBRARY_PATH | LIBRARY_PATH | C_INCLUDE_PATH | ldconfig

- GMP make install log

```
ldconfig -n $HOME/install/gmp/lib
-----------------------------------------------------------------
Libraries have been installed in:
   /home/mingzheng/install/gmp/lib

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the '-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the 'LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the 'LD_RUN_PATH' environment variable
     during linking
   - use the '-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to '/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
------------------------------------------------------------------
```

- MPFR/INSTALL

```
If you installed MPFR (header and library) in directories that are
not searched by default by the compiler and/or linking tools, then,
like with other libraries, you may need to set up some environment
variables such as C_INCLUDE_PATH (to find the header mpfr.h),
LIBRARY_PATH (to find the library), and if the shared library has
been installed, LD_LIBRARY_PATH (before execution) or LD_RUN_PATH
(before linking); this list is not exhaustive and some environment
variables may be specific to your system. "make install" gives some
instructions; please read them. You can also find more information
in the manuals of your compiler and linker. The MPFR FAQ may also
give some information.
```

- gcc make install

```
libtool: finish: PATH="/home/mingzheng/work/ninja:/home/mingzheng/install/re2c/bin:/home/mingzheng/install/openssl/bin:/home/mingzheng/install/cmake/bin:/home/mingzheng/.local/bin:/home/mingzheng/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin" ldconfig -n /home/mingzheng/install/gcc20230621/libexec/gcc/x86_64-linux-gnu/14.0.0
----------------------------------------------------------------------
Libraries have been installed in:
   /home/mingzheng/install/gcc20230621/libexec/gcc/x86_64-linux-gnu/14.0.0

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the `LD_RUN_PATH' environment variable
     during linking
   - use the `-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to `/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------

libtool: finish: PATH="/home/mingzheng/work/ninja:/home/mingzheng/install/re2c/bin:/home/mingzheng/install/openssl/bin:/home/mingzheng/install/cmake/bin:/home/mingzheng/.local/bin:/home/mingzheng/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin" ldconfig -n /home/mingzheng/install/gcc20230621/lib/../lib64
----------------------------------------------------------------------
Libraries have been installed in:
   /home/mingzheng/install/gcc20230621/lib/../lib64

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the `LD_RUN_PATH' environment variable
     during linking
   - use the `-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to `/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------

libtool: finish: PATH="/home/mingzheng/work/ninja:/home/mingzheng/install/re2c/bin:/home/mingzheng/install/openssl/bin:/home/mingzheng/install/cmake/bin:/home/mingzheng/.local/bin:/home/mingzheng/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin" ldconfig -n /home/mingzheng/install/gcc20230621/lib/gcc/x86_64-linux-gnu/14.0.0/plugin
----------------------------------------------------------------------
Libraries have been installed in:
   /home/mingzheng/install/gcc20230621/lib/gcc/x86_64-linux-gnu/14.0.0/plugin

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the `LD_RUN_PATH' environment variable
     during linking
   - use the `-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to `/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------


```

- openssl, python build

```
$ echo "/usr/local/openssl/lib" >> /etc/ld.so.conf
$ ldconfig
$
$ echo "/usr/local/python3/lib" >/etc/ld.so.conf.d/python3.conf
$ ldconfig
```

> https://xdouble.cn/archives/4

# How to Set, List and Remove Environment Variables in Linux
Environment variables are key-value pair in Linux which are stored permanently or temporarily to be used by applications through shell.
In this guide you are going to learn how to setup environment variables in Linux, list them and remove them.

The global environment variables are stored in etc/environment. Any changes that is made in this file reflects throughout the system for all users.

* Set Temporary Environment Variables
Temporary variables are only available to the current shell session. The variables will get deleted once you close the terminal.

You can create temporary variables using the following syntax.
```
KEY1=value
KEY2="value 2"
KEY3=value1:value2
```
- The environment variable names should be in UPPERCASE. They are case sensitive.
- The name and value pair should be separated by = sign without any spaces around it.
- Multiple values can be added to a single variable which is separated using colon:.
- The values that are having spaces should be enclosed using quotes " ".

* List Environment Variables
```
env
printenv
```

* Read Environment Variables
```
printenv HOME USERNAME
echo $HOME $USERNAME
```

* Delete Environment Variables
```
unset variablename
```
* Set Permanent Environment Variables
- /etc/environemnt: This file stores the variables that are globally accessible by all users throughout the system.
- /etc/profile: Whenever a bash shell in entered the variables in this file will gets loaded.
    To add environment variable to this file you need to use the export command.
- ~/.bashrc: User specific environment variables are added here.
    To load the added variables in your current session you need to use the source command. 
```
source ~/.bashrc
```

# Conclusion
Now you have learned how to set environment variables, list them and remove if not needed.

	-- from: https://www.cloudbooklet.com/how-to-set-list-and-remove-environment-variables-in-linux/
