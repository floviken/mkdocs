# Environment variables

## Environment variables

### Introduction

An environment variable is a named object that contains information used by one or more applications. By using environment variables one can easily change a configuration setting for one or more applications.

### Important examples

The following table lists a number of variables used by a Linux system and describes their use. Example values are presented after the table.

|Variable	|Description |
|-----------|------------|
|*PATH*	|This variable contains a colon-separated list of directories in which the system looks for executable files. If a name is entered of an executable (such as ls, rc-update, or emerge) but this executable is not located in a listed directory, then the system will not execute it (unless the full path is entered as the command, such as /bin/ls).|
|*ROOTPATH*	|This variable has the same function as PATH, but this one only lists the directories that should be checked when the root-user enters a command.|
|*LDPATH*	|This variable contains a colon-separated list of directories in which the dynamical linker searches through to find a library.|
|*MANPATH*	|This variable contains a colon-separated list of directories in which the man command searches for the man pages.|
|*INFODIR*	|This variable contains a colon-separated list of directories in which the info command searches for the info pages.|
|*PAGER*	|This variable contains the path to the program used to list the contents of files through (such as less or more).|
|*EDITOR*	|This variable contains the path to the program used to change the contents of files with (such as nano or vi).|
|*KDEDIRS*	|This variable contains a colon-separated list of directories which contain KDE-specific material.|
|*CONFIG_PROTECT*	|This variable contains a space-delimited list of directories which should be protected by Portage during package updates.|
|*CONFIG_PROTECT_MASK*	|This variable contains a space-delimited list of directories which should not be protected by Portage during package updates.|

Below is an example definition of all these variables:

```sh title="CODE Example settings for the mentioned variables"
PATH="/bin:/usr/bin:/usr/local/bin:/opt/bin:/usr/games/bin"
ROOTPATH="/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/sbin:/usr/local/bin"
LDPATH="/lib:/usr/lib:/usr/local/lib:/usr/lib/gcc-lib/i686-pc-linux-gnu/3.2.3"
MANPATH="/usr/share/man:/usr/local/share/man"
INFODIR="/usr/share/info:/usr/local/share/info"
PAGER="/usr/bin/less"
EDITOR="/usr/bin/vim"
KDEDIRS="/usr"
# Directories that are protected during package updates.
# Note the use of the \ (backslashes) on the end of the following lines which interprets to a single space-delimited line.
CONFIG_PROTECT="/usr/X11R6/lib/X11/xkb /opt/tomcat/conf \
                /usr/kde/3.1/share/config /usr/share/texmf/tex/generic/config/ \
                /usr/share/texmf/tex/platex/config/ /usr/share/config"
# Directories that are _not_ protected during package updates.
CONFIG_PROTECT_MASK="/etc/gconf"
```

## Defining variables globally

### The env.d directory

To centralize the definitions of these variables, Gentoo introduced the /etc/env.d/ directory. Inside this directory a number of files are available, such as 00basic, 05gcc, etc. which contain the variables needed by the application mentioned in their name.

For instance, when **gcc** is installed, a file called 05gcc was created by the ebuild which contains the definitions of the following variables:

```sh title="FILE /etc/env.d/05gccDefault gcc enabled environment variables"
PATH="/usr/i686-pc-linux-gnu/gcc-bin/3.2"
ROOTPATH="/usr/i686-pc-linux-gnu/gcc-bin/3.2"
MANPATH="/usr/share/gcc-data/i686-pc-linux-gnu/3.2/man"
INFOPATH="/usr/share/gcc-data/i686-pc-linux-gnu/3.2/info"
CC="gcc"
CXX="g++"
LDPATH="/usr/lib/gcc-lib/i686-pc-linux-gnu/3.2.3"
```

Other distributions might tell their users to change or add such environment variable definitions in /etc/profile or other locations. Gentoo on the other hand makes it easy for the user (and for Portage) to maintain and manage the environment variables without having to pay attention to the numerous files that can contain environment variables.

For instance, when **gcc** is updated, the /etc/env.d/05gcc file is updated too without requesting any user-interaction.

This not only benefits Portage, but also the user. Occasionally users might be asked to set a certain environment variable system-wide. As an example we take the *http_proxy* variable. Instead of messing about with /etc/profile, users can now just create a file (say /etc/env.d/99local) and enter the definition(s) in it:

```sh title="FILE /etc/env.d/99localSetting a global variable"
http_proxy="proxy.server.com:8080"
```

By using the same file for all self-managed variables, users have a quick overview on the variables they have defined themselves.

### env-update

Several files in /etc/env.d/ define the *PATH* variable. This is not a mistake: when the **env-update** command is executed, it will append the several definitions before it updates the environment variables, thereby making it easy for packages (or users) to add their own environment variable settings without interfering with the already existing values.

The **env-update** script will append the values in the alphabetical order of the /etc/env.d/ files. The file names must begin with two decimal digits.

```sh title="CODE Update order used by env-update"
00basic        99kde-env       99local
     +-------------+----------------+-------------+
PATH="/bin:/usr/bin:/usr/kde/3.2/bin:/usr/local/bin"
```

The concatenation of variables does not always happen, only with the following variables: *ADA_INCLUDE_PATH, ADA_OBJECTS_PATH, CLASSPATH, KDEDIRS, PATH, LDPATH, MANPATH, INFODIR, INFOPATH, ROOTPATH, CONFIG_PROTECT, CONFIG_PROTECT_MASK, PRELINK_PATH, PRELINK_PATH_MASK, PKG_CONFIG_PATH*, and *PYTHONPATH*. For all other variables the latest defined value (in alphabetical order of the files in /etc/env.d/) is used.

It is possible to add more variables into this list of concatenate-variables by adding the variable name to either *COLON_SEPARATED* or *SPACE_SEPARATED* variables (also inside an /etc/env.d/ file).

When executing **env-update**, the script will create all environment variables and place them in /etc/profile.env (which is used by /etc/profile). It will also extract the information from the *LDPATH* variable and use that to create /etc/ld.so.conf. After this, it will run *ldconfig* to recreate the /etc/ld.so.cache file used by the dynamical linker.

To notice the effect of **env-update** immediately after running it, execute the following command to update the environment. Users who have installed Gentoo themselves will probably remember this from the installation instructions:

`root # env-update && source /etc/profile`

!!! Note
```The above command only updates the variables in the current terminal, new consoles, and their children. Thus, if the user is working in X11, he needs to either type source /etc/profile in every new terminal opened or restart X so that all new terminals source the new variables. If a login manager is used, it is necessary to become root and restart the /etc/init.d/xdm service.
```

!!! Important
```It is not possible to use shell variables when defining other variables. This means things like FOO="$BAR" (where $BAR is another variable) are forbidden.
```

## Defining variables locally

### User specific

It might not be necessary to define an environment variable globally. For instance, one might want to add /home/my_user/bin and the current working directory (the directory the user is in) to the *PATH* variable but do not want all other users on the system to have that in their *PATH* too. To define an environment variable locally, use ~/.bashrc or ~/.bash_profile:

```sh title="FILE ~/.bashrcExtending PATH for local usage"
# A colon followed by no directory is treated as the current working directory
PATH="${PATH}:/home/my_user/bin:"
```

After logout/login, the *PATH* variable will be updated.

### Session specific

Sometimes even stricter definitions are requested. For instance, a user might want to be able to use binaries from a temporary directory created without using the path to the binaries themselves or editing ~/.bashrc for the short time necessary.

In this case, just define the PATH variable in the current session by using the export command. As long as the user does not log out, the PATH variable will be using the temporary settings.

`root #export PATH="${PATH}:/home/my_user/tmp/usr/bin"`

