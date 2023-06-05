


# Runlevels

## Runlevels

### Booting the system

When the system is booted, lots of text floats by. When paying close attention, one will notice this text is (usually) the same every time the system is rebooted. The sequence of all these actions is called the boot sequence and is (more or less) statically defined.

First, the boot loader will load the kernel image that is defined in the boot loader configuration. Then, the boot loader instructs the CPU to execute kernel. When the kernel is loaded and run, it initializes all kernel-specific structures and tasks and starts the init process.

This process then makes sure that all filesystems (defined in /etc/fstab) are mounted and ready to be used. Then it executes several scripts located in /etc/init.d/, which will start the services needed in order to have a successfully booted system.

Finally, when all scripts are executed, init activates the terminals (in most cases just the virtual consoles which are hidden beneath `Alt+F1`, `Alt+F2`, etc.) attaching a special process called agetty to it. This process will then make sure users are able to log on through these terminals by running login.

Initscripts
Now init doesn't just execute the scripts in /etc/init.d/ randomly. Even more, it doesn't run all scripts in /etc/init.d/, only the scripts it is told to execute. It decides which scripts to execute by looking into /etc/runlevels/.

First, init runs all scripts from /etc/init.d/ that have symbolic links inside /etc/runlevels/boot/. Usually, it will start the scripts in alphabetical order, but some scripts have dependency information in them, telling the system that another script must be run before they can be started.

When all /etc/runlevels/boot/ referenced scripts are executed, init continues with running the scripts that have a symbolic link to them in /etc/runlevels/default/. Again, it will use the alphabetical order to decide what script to run first, unless a script has dependency information in it, in which case the order is changed to provide a valid start-up sequence. The latter is also the reason why commands used during the installation of Gentoo Linux used `default`, as in **rc-update add sshd default**.

### How init works

Of course init doesn't decide all that by itself. It needs a configuration file that specifies what actions need to be taken. This configuration file is /etc/inittab.

Remember the boot sequence that was just described - init's first action is to mount all file systems. This is defined in the following line from /etc/inittab:

``` sh title="FILE /etc/inittabInitialization command"
si::sysinit:/sbin/openrc sysinit
``` 

This line tells init that it must run **/sbin/openrc sysinit** to initialize the system. The /sbin/openrc script takes care of the initialization, so one might say that init doesn't do much - it delegates the task of initializing the system to another process.

Second, init executed all scripts that had symbolic links in /etc/runlevels/boot/. This is defined in the following line:

```sh title="FILE /etc/inittabBoot command invocation"
rc::bootwait:/sbin/openrc boot
```

Again the openrc script performs the necessary tasks. Note that the option given to openrc (boot) is the same as the subdirectory of /etc/runlevels/ that is used.

Now init checks its configuration file to see what runlevel it should run. To decide this, it reads the following line from /etc/inittab:

```sh title="FILE /etc/inittabDefault runlevel selection"
id:3:initdefault:
```

In this case (which the majority of Gentoo users will use), the runlevel id is 3. Using this information, init checks what it must run to start runlevel 3:

```sh title="FILE /etc/inittabRunlevel definitions"
l0:0:wait:/sbin/openrc shutdown
l1:S1:wait:/sbin/openrc single
l2:2:wait:/sbin/openrc nonetwork
l3:3:wait:/sbin/openrc default
l4:4:wait:/sbin/openrc default
l5:5:wait:/sbin/openrc default
l6:6:wait:/sbin/openrc reboot
```

The line that defines level 3, again, uses the openrc script to start the services (now with argument `default`). Again note that the argument of openrc is the same as the subdirectory from /etc/runlevels/.

When openrc has finished, init decides what virtual consoles it should activate and what commands need to be run at each console:

```sh title="FILE /etc/inittabTerminal definitions"
c1:12345:respawn:/sbin/agetty 38400 tty1 linux
c2:12345:respawn:/sbin/agetty 38400 tty2 linux
c3:12345:respawn:/sbin/agetty 38400 tty3 linux
c4:12345:respawn:/sbin/agetty 38400 tty4 linux
c5:12345:respawn:/sbin/agetty 38400 tty5 linux
c6:12345:respawn:/sbin/agetty 38400 tty6 linux
```

### Available runlevels

In a previous section, we saw that init uses a numbering scheme to decide what runlevel it should activate. A runlevel is a state in which the system is running and contains a collection of scripts (runlevel scripts or initscripts) that must be executed when entering or leaving a runlevel.

In Gentoo, there are seven runlevels defined: three internal runlevels, and four user-defined runlevels. The internal runlevels are called sysinit, shutdown and reboot and do exactly what their names imply: initialize the system, powering off the system, and rebooting the system.

The user-defined runlevels are those with an accompanying /etc/runlevels/ subdirectory: boot, default, nonetwork and single. The boot runlevel starts all system-necessary services which all other runlevels use. The remaining three runlevels differ in what services they start: default is used for day-to-day operations, nonetwork is used in case no network connectivity is required, and single is used when the system needs to be fixed.

### Working with initscripts

The scripts that the openrc process starts are called init scripts. Each script in /etc/init.d/ can be executed with the arguments `start`, `stop`, `restart`, `zap`, `status`, `ineed`, `iuse`, `iwant`, `needsme`, `usesme`, or wantsme.

To start, stop, or restart a service (and all depending services), the `start`, `stop`, and `restart` arguments should be used:

`root # rc-service postfix start`

!!! note

``` markdown 
Only the services that need the given service are stopped or restarted. 
The other depending services (those that use the service but don't need it) 
are left untouched.
```

To stop a service, but not the services that depend on it, use the `--nodeps` option together with the `stop` argument:

`root # rc-service --nodeps postfix stop`

To see what status a service has (started, stopped, ...) use the `status` argument:

`root # rc-service postfix status`

If the status information shows that the service is running, but in reality it is not, then reset the status information to "stopped" with the `zap` argument:

`root # rc-service postfix zap`

To also ask what dependencies the service has, use `iwant`, `iuse` or `ineed`. With `ineed` it is possible to see the services that are really necessary for the correct functioning of the service. `iwant` or `iuse`, on the other hand, shows the services that can be used by the service, but are not necessary for the correct functioning.

`root # rc-service postfix ineed`

Similarly, it is possible to ask what services require the service (`needsme`) or can use it (`usesme` or `wantsme`):

`root # rc-service postfix needsme`

##  Updating runlevels

### rc-update

Gentoo's init system uses a dependency-tree to decide what service needs to be started first. As this is a tedious task that we wouldn't want our users to have to do manually, we have created tools that ease the administration of the runlevels and init scripts.

With **rc-update** it is possible to add and remove init scripts to a runlevel. The **rc-update** tool will then automatically ask the depscan.sh script to rebuild the dependency tree.

### Adding and removing services

In earlier instructions, init scripts have already been added to the "default" runlevel. What "default" means has been explained earlier in this document. Next to the runlevel, the **rc-update** script requires a second argument that defines the action: `add`, `del`, or `show`.

To add or remove an init script, just give **rc-update** the add or del argument, followed by the init script and the runlevel. For instance:

`root # rc-update del postfix default`

The **rc-update -v show** command will show all the available init scripts and list at which runlevels they will execute:

`root # rc-update -v show`

It is also possible to run **rc-update show** (without -v) to just view enabled init scripts and their runlevels.

## Configuring services

### Why additional configuration is needed

Init scripts can be quite complex. It is therefore not really desirable to have the users edit the init script directly, as it would make it more error-prone. It is however important to be able to configure such a service. For instance, users might want to give more options to the service itself.

A second reason to have this configuration outside the init script is to be able to update the init scripts without the fear that the user's configuration changes will be undone.

### conf.d directory

Gentoo provides an easy way to configure such a service: every init script that can be configured has a file in /etc/conf.d/. For instance, the apache2 initscript (called /etc/init.d/apache2) has a configuration file called /etc/conf.d/apache2, which can contain the options to give to the Apache 2 server when it is started:

```sh title="FILE /etc/conf.d/apache2Example options for apache2 init script"
APACHE2_OPTS="-D PHP5"
```

Such a configuration file contains only variables (just like /etc/portage/make.conf does), making it very easy to configure services. It also allows us to provide more information about the variables (as comments).

### Writing initscripts
Another useful resource is OpenRC's [service script guide](https://github.com/OpenRC/openrc/blob/master/service-script-guide.md).

### Is it necessary?

No, writing an init script is usually not necessary as Gentoo provides ready-to-use init scripts for all provided services. However, some users might have installed a service without using Portage, in which case they will most likely have to create an init script.

Do not use the init script provided by the service if it isn't explicitly written for Gentoo: Gentoo's init scripts are not compatible with the init scripts used by other distributions! That is, unless the other distribution is using [OpenRC](https://wiki.gentoo.org/wiki/OpenRC)!

### Layout
The basic layout of an init script is shown below.

``` sh title="CODE Example initscript layout (traditional)"
#!/sbin/openrc-run
  
depend() {
#  (Dependency information)
}
  
start() {
#  (Commands necessary to start the service)
}
  
stop() {
#  (Commands necessary to stop the service)
}
```


``` sh title="CODE Example initscript layout (updated)"
#!/sbin/openrc-run
command=/usr/bin/foo
command_args="${foo_args} --bar"
pidfile=/var/run/foo.pid
name="FooBar Daemon"
 
description="FooBar is a daemon that drinks"
extra_started_commands="drink"
description_drink="Opens mouth and reflexively swallows"
 
depend() {
#  (Dependency information)
}
 
start_pre() {
#  (Commands necessary to prepare to start the service)
    # Ensure that our dirs are correct
    checkpath --directory --owner foo:foo --mode 0775 \
        /var/run/foo /var/cache/foo
}
  
stop_post() {
#  (Commands necessary to clean up after the service)
    # Clean any spills
    rm -rf /var/cache/foo/*
}
 
drink() {
    ebegin "Starting to drink"
    ${command} --drink beer
    eend $? "Failed to drink any beer :("
}

``` 

Every init script requires the `start()` function or `command` variable to be defined. All other sections are optional.

### Dependencies

There are three dependency-alike settings that can be defined which influence the start-up or sequencing of init scripts: `want`, `use` and `need`. Next to these, there are also two order-influencing methods called `before` and `after`. These last two are no dependencies per se - they do not make the original init script fail if the selected one isn't scheduled to start (or fails to start).

- The `use` settings informs the init system that this script uses functionality offered by the selected script, but does not directly depend on it. A good example would be `use logger` or `use dns`. If those services are available, they will be put in good use, but if the system does not have a logger or DNS server the services will still work. If the services exist, then they are started before the script that uses them.

- The `want` setting is similar to `use` with one exception. `use` only considers services which were added to an init level. `want` will try to start any available service even if not added to an init level.

- The `need` setting is a hard dependency. It means that the script that is needing another script will not start before the other script is launched successfully. Also, if that other script is restarted, then this one will be restarted as well.

- When using `before`, then the given script is launched before the selected one if the selected one is part of the init level. So an init script xdm that defines `before alsasound` will start before the alsasound script, but only if alsasound is scheduled to start as well in the same init level. If alsasound is not scheduled to start too, then this particular setting has no effect and xdm will be started when the init system deems it most appropriate.

- Similarly, `after` informs the init system that the given script should be launched after the selected one if the selected one is part of the init level. If not, then the setting has no effect and the script will be launched by the init system when it deems it most appropriate.

It should be clear from the above that `need` is the only "true" dependency setting as it affects if the script will be started or not. All the others are merely pointers towards the init system to clarify in which order scripts can be (or should be) launched.

Now, look at many of Gentoo's available init scripts and notice that some have dependencies on things that are no init scripts. These "things" we call virtuals.

A *virtual dependency* is a dependency that a service provides, but that is not provided solely by that service. An init script can depend on a system logger, but there are many system loggers available (metalogd, syslog-ng, sysklogd, ...). As the script cannot need every single one of them (no sensible system has all these system loggers installed and running) we made sure that all these services provide a virtual dependency.

For instance, take a look at the postfix dependency information:

```sh title="FILE /etc/init.d/postfixDependency information of the postfix service"
depend() {
  need net
  use logger dns
  provide mta
}
```

As can be seen, the postfix service:

- Requires the (virtual) net dependency (which is provided by, for instance, /etc/init.d/net.eth0).
- Uses the (virtual) logger dependency (which is provided by, for instance, /etc/init.d/syslog-ng).
- Uses the (virtual) dns dependency (which is provided by, for instance, /etc/init.d/named).
- Provides the (virtual) mta dependency (which is common for all mail servers).

### Controlling the order

As described in the previous section, it is possible to tell the init system what order it should use for starting (or stopping) scripts. This ordering is handled both through the dependency settings use and need, but also through the order settings before and after. As we have described these earlier already, let's take a look at the portmap service as an example of such init script.

```sh title="FILE /etc/init.d/portmapDependency information of the portmap service"
depend() {
  need net
  before inetd
  before xinetd
}
```
It is possible to use the "*" glob to catch all services in the same runlevel, although this isn't advisable.

```sh title="CODE Using the * glob"
depend() {
  before *
}
```

If the service must write to local disks, it should need localmount. If it places anything in /var/run/ such as a pidfile, then it should start after bootmisc:

```sh title="CODE Dependency setting with needing localmount and after bootmisc"
depend() {
  need localmount
  after bootmisc
}
```

### Standard functions

Next to the `depend()` functionality, it is also necessary to define the `start()` function. This one contains all the commands necessary to initialize the service. It is advisable to use the `ebegin` and `eend` functions to inform the user about what is happening:

```sh title="CODE Example start() function"
start() {
  if [ "${RC_CMD}" = "restart" ];
  then
    # Do something in case a restart requires more than stop, start
  fi
  
  ebegin "Starting my_service"
  start-stop-daemon --start --exec /path/to/my_service \
    --pidfile /path/to/my_pidfile
  eend $?
}
```

Both `--exec` and `--pidfile` should be used in start and stop functions. If the service does not create a pidfile, then use `--make-pidfile` if possible, though it is recommended to test this to be sure. Otherwise, don't use pidfiles. It is also possible to add `--quiet` to the start-stop-daemon options, but this is not recommended unless the service is extremely verbose. Using `--quiet` may hinder debugging if the service fails to start.

Another notable setting used in the above example is to check the contents of the *RC_CMD* variable. Unlike the previous init script system, the newer OpenRC system does not support script-specific restart functionality. Instead, the script needs to check the contents of the *RC_CMD* variable to see if a function (be it `start()` or `stop()`) is called as part of a restart or not.

!!! Note
```
Make sure that `--exec` actually calls a service and not just a shell script that launches services and exits - that's what the init script is supposed to do.
```

For more examples of the `start()` function, please read the source code of the available init scripts in the /etc/init.d/ directory.

Another function that can (but does not have to) be defined is `stop()`. The init system is intelligent enough to fill in this function by itself if start-stop-daemon is used.

```sh title="CODE Example stop() function"
stop() {
  ebegin "Stopping my_service"
  start-stop-daemon --stop --exec /path/to/my_service \
    --pidfile /path/to/my_pidfile
  eend $?
}
```

If the service runs some other script (for example, Bash, Python, or Perl), and this script later changes names (for example, foo.py to foo), then it is necessary to add `--name` to start-stop-daemon. This must specify the name that the script will be changed to. In this example, a service starts foo.py, which changes names to foo:


```sh title=""CODE Example definition for a service that starts the foo script"
start() {
  ebegin "Starting my_script"
  start-stop-daemon --start --exec /path/to/my_script \
    --pidfile /path/to/my_pidfile --name foo
  eend $?
}
```

start-stop-daemon has an excellent man page available if more information is needed:

`user $ man start-stop-daemon`

Gentoo's init script syntax is based on the POSIX Shell so people are free to use sh-compatible constructs inside their init scripts. Keep other constructs, like bash-specific ones, out of the init scripts to ensure that the scripts remain functional regardless of the change Gentoo might do on its init system.

### Adding custom options

If the initscript needs to support more options than the ones we have already encountered, then add the option to one of the following variables, and create a function with the same name as the option. For instance, to support an option called `restartdelay`:

- *extra_commands* - Command is available with the service in any state
- *extra_started_commands* - Command is available when the service is started
- *extra_stopped_commands* - Command is available when the service is stopped

```sh title="CODE Example definition of restartdelay method"
extra_started_commands="restartdelay"
  
restartdelay() {
  stop
  sleep 3    # Wait 3 seconds before starting again
  start
}
```

!!! Important
The `restart()` function cannot be overridden in OpenRC!

### Service configuration variables

In order to support configuration files in /etc/conf.d/, no specifics need to be implemented: when the init script is executed, the following files are automatically sourced (i.e. the variables are available to use):

- /etc/conf.d/YOUR_INIT_SCRIPT
- /etc/conf.d/basic
- /etc/rc.conf

Also, if the init script provides a virtual dependency (such as net), the file associated with that dependency (such as /etc/conf.d/net) will be sourced too.

## Changing runlevel behavior

### Who might benefit

Many laptop users know the situation: at home they need to start net.eth0, but they don't want to start net.eth0 while on the road (as there is no network available). With Gentoo the runlevel behaviour can be altered at will.

For instance, a second "default" runlevel can be created which can be booted that has other init scripts assigned to it. At boottime, the user can then select what default runlevel to use.

### Using softlevel

First of all, create the runlevel directory for the second "default" runlevel. As an example we create the *offline* runlevel:

`root # mkdir /etc/runlevels/offline`

Add the necessary init scripts to the newly created runlevel. For instance, to have an exact copy of the current default runlevel but without net.eth0:

```
root # cd /etc/runlevels/default
root # for service in *; do rc-update add $service offline; done
root # rc-update del net.eth0 offline
root # rc-update show offline
(Partial sample Output)
               acpid | offline
          domainname | offline
               local | offline
            net.eth0 |
```
Even though net.eth0 has been removed from the offline runlevel, udev might want to attempt to start any devices it detects and launch the appropriate services, a functionality that is called hotplugging. By default, Gentoo does not enable hotplugging.

To enable hotplugging, but only for a selected set of scripts, use the *rc_hotplug* variable in /etc/rc.conf:

```sh title=""FILE /etc/rc.confEnable hotplugging of the WLAN interface"
rc_hotplug="net.wlan !net.*"
``` 

!!! Note
For more information on device initiated services, please see the comments inside /etc/rc.conf.
Edit the bootloader configuration and add a new entry for the offline runlevel. In that entry, add softlevel=offline as a boot parameter.

### Using bootlevel
Using bootlevel is completely analogous to softlevel. The only difference here is that a second "boot" runlevel is defined instead of a second "default" runlevel.





Environment variables
Introduction
An environment variable is a named object that contains information used by one or more applications. By using environment variables one can easily change a configuration setting for one or more applications.

Important examples
The following table lists a number of variables used by a Linux system and describes their use. Example values are presented after the table.

Variable	Description
PATH	This variable contains a colon-separated list of directories in which the system looks for executable files. If a name is entered of an executable (such as ls, rc-update, or emerge) but this executable is not located in a listed directory, then the system will not execute it (unless the full path is entered as the command, such as /bin/ls).
ROOTPATH	This variable has the same function as PATH, but this one only lists the directories that should be checked when the root-user enters a command.
LDPATH	This variable contains a colon-separated list of directories in which the dynamical linker searches through to find a library.
MANPATH	This variable contains a colon-separated list of directories in which the man command searches for the man pages.
INFODIR	This variable contains a colon-separated list of directories in which the info command searches for the info pages.
PAGER	This variable contains the path to the program used to list the contents of files through (such as less or more).
EDITOR	This variable contains the path to the program used to change the contents of files with (such as nano or vi).
KDEDIRS	This variable contains a colon-separated list of directories which contain KDE-specific material.
CONFIG_PROTECT	This variable contains a space-delimited list of directories which should be protected by Portage during package updates.
CONFIG_PROTECT_MASK	This variable contains a space-delimited list of directories which should not be protected by Portage during package updates.
Below is an example definition of all these variables:

CODE Example settings for the mentioned variables
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
Defining variables globally
The env.d directory
To centralize the definitions of these variables, Gentoo introduced the /etc/env.d/ directory. Inside this directory a number of files are available, such as 00basic, 05gcc, etc. which contain the variables needed by the application mentioned in their name.

For instance, when gcc is installed, a file called 05gcc was created by the ebuild which contains the definitions of the following variables:

FILE /etc/env.d/05gccDefault gcc enabled environment variables
PATH="/usr/i686-pc-linux-gnu/gcc-bin/3.2"
ROOTPATH="/usr/i686-pc-linux-gnu/gcc-bin/3.2"
MANPATH="/usr/share/gcc-data/i686-pc-linux-gnu/3.2/man"
INFOPATH="/usr/share/gcc-data/i686-pc-linux-gnu/3.2/info"
CC="gcc"
CXX="g++"
LDPATH="/usr/lib/gcc-lib/i686-pc-linux-gnu/3.2.3"
Other distributions might tell their users to change or add such environment variable definitions in /etc/profile or other locations. Gentoo on the other hand makes it easy for the user (and for Portage) to maintain and manage the environment variables without having to pay attention to the numerous files that can contain environment variables.

For instance, when gcc is updated, the /etc/env.d/05gcc file is updated too without requesting any user-interaction.

This not only benefits Portage, but also the user. Occasionally users might be asked to set a certain environment variable system-wide. As an example we take the http_proxy variable. Instead of messing about with /etc/profile, users can now just create a file (say /etc/env.d/99local) and enter the definition(s) in it:

FILE /etc/env.d/99localSetting a global variable
http_proxy="proxy.server.com:8080"
By using the same file for all self-managed variables, users have a quick overview on the variables they have defined themselves.

env-update
Several files in /etc/env.d/ define the PATH variable. This is not a mistake: when the env-update command is executed, it will append the several definitions before it updates the environment variables, thereby making it easy for packages (or users) to add their own environment variable settings without interfering with the already existing values.

The env-update script will append the values in the alphabetical order of the /etc/env.d/ files. The file names must begin with two decimal digits.

CODE Update order used by env-update
00basic        99kde-env       99local
     +-------------+----------------+-------------+
PATH="/bin:/usr/bin:/usr/kde/3.2/bin:/usr/local/bin"
The concatenation of variables does not always happen, only with the following variables: ADA_INCLUDE_PATH, ADA_OBJECTS_PATH, CLASSPATH, KDEDIRS, PATH, LDPATH, MANPATH, INFODIR, INFOPATH, ROOTPATH, CONFIG_PROTECT, CONFIG_PROTECT_MASK, PRELINK_PATH, PRELINK_PATH_MASK, PKG_CONFIG_PATH, and PYTHONPATH. For all other variables the latest defined value (in alphabetical order of the files in /etc/env.d/) is used.

It is possible to add more variables into this list of concatenate-variables by adding the variable name to either COLON_SEPARATED or SPACE_SEPARATED variables (also inside an /etc/env.d/ file).

When executing env-update, the script will create all environment variables and place them in /etc/profile.env (which is used by /etc/profile). It will also extract the information from the LDPATH variable and use that to create /etc/ld.so.conf. After this, it will run ldconfig to recreate the /etc/ld.so.cache file used by the dynamical linker.

To notice the effect of env-update immediately after running it, execute the following command to update the environment. Users who have installed Gentoo themselves will probably remember this from the installation instructions:

root #env-update && source /etc/profile
 Note
The above command only updates the variables in the current terminal, new consoles, and their children. Thus, if the user is working in X11, he needs to either type source /etc/profile in every new terminal opened or restart X so that all new terminals source the new variables. If a login manager is used, it is necessary to become root and restart the /etc/init.d/xdm service.
 Important
It is not possible to use shell variables when defining other variables. This means things like FOO="$BAR" (where $BAR is another variable) are forbidden.
Defining variables locally
User specific
It might not be necessary to define an environment variable globally. For instance, one might want to add /home/my_user/bin and the current working directory (the directory the user is in) to the PATH variable but do not want all other users on the system to have that in their PATH too. To define an environment variable locally, use ~/.bashrc or ~/.bash_profile:

FILE ~/.bashrcExtending PATH for local usage
# A colon followed by no directory is treated as the current working directory
PATH="${PATH}:/home/my_user/bin:"
After logout/login, the PATH variable will be updated.

Session specific
Sometimes even stricter definitions are requested. For instance, a user might want to be able to use binaries from a temporary directory created without using the path to the binaries themselves or editing ~/.bashrc for the short time necessary.

In this case, just define the PATH variable in the current session by using the export command. As long as the user does not log out, the PATH variable will be using the temporary settings.

root #export PATH="${PATH}:/home/my_user/tmp/usr/bin"

