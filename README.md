pgenv - PostgreSQL binary manager
=================================

Synopsis
--------

    pgenv help

    # Check dependencies
    pgenv check

    # Show versions available to build
    pgenv available

    # Build PostgreSQL server
    pgenv build 10.4

    # Switch PostgreSQL version
    pgenv switch 10.4
    
    # Use PostgreSQL version
    pgenv use 10.4
    
    # Stop current version
    pgenv stop
    
    # Start current version
    pgenv start

    # Show server status
    pgenv status

    # Restart current version
    pgenv restart
   
    # Show current version
    pgenv version
    
    # List built versions
    pgenv versions

    # Clear current version
    pgenv clear

    # Remove PostgreSQL version
    pgenv remove 8.0.25

Description
-----------

pgenv is a simple utility to build and run different releases of [PostgreSQL].
This makes it easy to switch between versions when testing applications for
compatibility.

Installation
------------

1.  Check out pgenv into `~/.pgenv`.

    ``` sh
    git clone https://github.com/theory/pgenv.git ~/.pgenv
    ```

2.  Add `~/.pgenv/bin` and `~/.pgenv/pgsql/bin` to your `$PATH` for access to
    the `pgenv` command-line utility and all the programs provided by
    PostgreSQL:
   
    ``` sh
    echo 'export PATH="$HOME/.pgenv/bin:$HOME/.pgenv/pgsql/bin:$PATH"' >> ~/.bash_profile
    ```

    **Ubuntu note**: Modify your `~/.profile` instead of `~/.bash_profile`.

    **Zsh note**: Modify your `~/.zshrc` file instead of `~/.bash_profile`.

3.  Restart your shell as a login shell so the path changes take effect. You
    can now begin using pgenv.

    ``` sh
    exec $SHELL -l
    ```

4.  Build a version of PostgreSQL:

    ``` sh
    pgenv build 10.4
    ```

### Configuration

By default, all versions of PostgreSQL will be built in the root of the
project directory (generally in `~/.pgenv`.). If you'd like them to live
elsewhere, set the `$PGENV_ROOT` environment variable to the appropriate
directory.

It is possible to configure which programs, flags and languages to build
using a configuration file before the program launches the build. 
For a more detailed configuration, see the [`pgenv config`](#pgenv-config)
command below.

You can use a local PostgreSQL git repo instead of downloading tarballs
for the build step by setting the external `$PGENV_LOCAL_POSTGRESQL_REPO`
environment variable to the appropriate absolute path.

### Running scripts

It is possible to run custom script when particular events happen. 
The purpose is to let the user to configure the cluster as much as
she want, for instance installing extension or pre-loading data.

Custom scripts are driven by a set of configuration variables as follows:
- `PGENV_SCRIPT_POSTINSTALL` is an executable script run as soon as the `build`
finishes. The return value of the script does not affect the `pgenv` workflow.
The script gets the version of PostgreSQL being installed as first argument.

-   `PGENV_SCRIPT_POSTINITDB` is an executable script run as soon as the `use`
    command ends the `initdb` phase, that happens only the `PGDATA` has not been
    initialized (i.e., on *very first `use`*). The return value of the script
    does not affect the `pgenv` workflow. The script gets the current `PGDATA`
    path as first argument.

-   `PGENV_SCRIPT_POSTSTART` is an executable script executed each time an
    instances is started, that is `start` is executed. The return value of the
    script does not affect the `pgenv` workflow. The script gets the current
    `PGDATA` path as first argument.

-   `PGENV_SCRIPT_PRESTOP` is an executable script executed before an instance
    is stopped, that is before the `stop` command is executed. The return value
    of the script does not affect the `pgenv` workflow, and if the script
    generates errors they are ignored, so that the cluster is going to be
    stopped anyway. The script gets the current `PGDATA` path as first argument.

-   `PGENV_SCRIPT_POSTSTOP` is an executable script executed each time an
    instance is stopped, that is the `stop` command is executed. The return
    value of the script does not affect the `pgenv` workflow. The script gets
    the current `PGDATA` path as first argument.

-   `PGENV_SCRIPT_POSTRESTART` is an executable script executed each time an
    instance is restarted, that is the `restart` command is executed. The return
    value of the script does not affect the `pgenv` workflow. The script gets
    the current `PGDATA` path as first argument.

The above configuration variables can be set in the configuration file or on the
fly, for instance:

``` sh
$ export PGENV_SCRIPT_POSTSTART=/usr/local/bin/load_data.sh
$ pgenv start 12.0
```

It is worth noting that the start, restart and post script are not shared
between similar events: for instance the `PGENV_SCRIPT_POSTSTOP` is not executed
if the user issues a `restart` command (even if that implies somehow a cluster
stop).

**Please note that running external scripts can result in damages and crashes in
the `pgenv` workflow.** In the future, more hooks to run scripts could be added.

### Upgrading

You can upgrade your installation to the cutting-edge version at any time
with a simple `git pull`.

``` sh
$ cd ~/.pgenv
$ git pull
```

### Dependencies
------------

*   env - Sets environment for execution
*   bash - Command shell interpreter
*   curl - Used to download files
*   sed, grep, cat, tar, sort, tr, uname, tail - General Unix command line utilities
*   patch - For patching versions that need patching
*   make -  Builds PostgreSQL

Optional dependencies:

*   Perl 5 - To build PL/Perl 
*   Python - To build PL/Python

Command Reference
-----------------

Like `git`, the `pgenv` command delegates to subcommands based on its
first argument. 

Some commands require you to specify the PostgreSQL version to act on.
You can specify the version the command applies to by either entering the
PostgreSQL version number or by specifying any of the special keywords:

-    `current` or `version` to indicate the currently selected PostgreSQL version;
-    `earliest` to indicate the oldest installed version (excluding beta versions);
-    `newest` to indicate the newest installed version (excluding beta versions).

It is important to note that `earliest` and `latest` have nothing to do with the
time you installed PostgreSQL by means of `pgenv`: they refer only to PostgreSQL
*stable* versions. To better clarify this, the following snippet shows you which
aliases point to which versions in an example installation.

```
9.6.19                 <-- earliest (also earliest 9.6)
9.6.20      <-- latest 9.6
12.2        <-- earliest 12
12.4        <-- latest 12
13beta2
13.0                   <-- latest (also latest 13)
```

The subcommands are:

### pgenv use

Sets the version of PostgreSQL to be used in all shells by symlinking its
directory to `~/$PGENV_ROOT/pgsql` and starting it. Initializes the data
directory if none exists. If another version is currently active, it will be
stopped before switching. If the specified version is already in use, the `use`
command won't stop it, but will initialize its data directory and starts it if
it's not already running.

```
$ pgenv use 10.4
waiting for server to shut down.... done
server stopped
waiting for server to start.... done
server started
PostgreSQL 10.4 started
```

The `use` command supports the special keywords `earliest` and `latest` to
respectively indicate the oldest PostgreSQL version installed and the newest
one. It is also possible to indicate a major version to narrow the scope of the
special keywords. As an example:

``` sh
pgenv use latest 10
```

will select the most recent PostgreSQL version of the 10 series installed.    

### pgenv switch

Sets the version of PostgreSQL to be used in all shells by symlinking its
directory to `$PGENV_ROOT/pgsql`. Contrary to `pgenv use` this command does
_not_ manage a database for you. Meaning, it will not start, stop and
initialize a postgres database with the given version. Instead it simply
changes the environment to a different version of PostgreSQL. This can be
useful if the user has other tools to automate the provisioning and lifecycle
of a database.

```
$ pgenv switch 10.4
```

### pgenv versions

Lists all PostgreSQL versions known to `pgenv`, and shows an asterisk next to
the currently active version (if any). The first column reports versions
available for use by `pgenv` and the second lists the subdirectory of
`$PGENV_ROOT` in which the each version is installed:

```
$ pgenv versions
      10.4      pgsql-10.4
      11beta3   pgsql-11beta3
      9.5.13    pgsql-9.5.13
  *   9.6.9     pgsql-9.6.9
```

In this example, versions `9.5.13`, `9.6.9`, `10.4`, and `11beta3` are available
for use, and the `*` indicates that `9.6.10` is the currently active version.
Each version is installed in a `pgsql-` subdirectory of `$PGENV_ROOT`.

### pgenv current

Displays the currently active PostgreSQL version.

```
$ pgenv current
10.4
```

Please note that `version` is a command synonym for version:

```
$ pgenv version
10.4
```

### pgenv clear

Clears the currently active version of PostgreSQL. If the current version is
running, `clear` will stop it before clearing it.

```
$ pgenv clear
waiting for server to shut down.... done
server stopped
PostgreSQL stopped
PostgreSQL cleared
```

### pgenv build

Downloads and builds the specified version of PostgreSQL and its contrib
modules, as far back as version `8.0`. It is possible to instrument the build
process to patch the source tree, see the section on patching later on. If the
version is already built, it will not be rebuilt; use `clear` to remove an
existing version before building it again.

```
$ pgenv build 10.3
# [Curl, configure, and make output elided]
PostgreSQL 10.3 built
```

The build phase can be customized via a configuration file, in the case the
system does not find a configuration file when the `build` is executed, a
warning is shown to the user to remind she can edit a configuration file and
start over the build process:

```
$ pgenv build 10.3
  ...
WARNING: no configuration file found for version 10.3
HINT: if you wish to customize the build process please
stop the execution within 5 seconds (CTRL-c) and run
    pgenv config write 10.3 && pgenv config edit 10.3
adjust 'configure' and 'make' options and flags and run again
    pgenv build 10.3
```

Within the configuration file it is possible to instrument the build phase from
the configuration to the actual build. For instance, in order to build with
PL/Perl, it is possible to configure the variable `PGENV_CONFIGURE_OPTIONS`
adding `--with-perl`. Or say you need SSL support and to tell teh compiler to
use Homebrew-installed OpenSSL. Edit it something like:

``` 
PGENV_CONFIGURE_OPTIONS=(
    --with-perl
    --with-openssl
    'CFLAGS=-I/opt/local/opt/openssl/include -I/opt/local/opt/libxml2/include'
    'LDFLAGS=-L/opt/local/opt/openssl/lib -L/opt/local/opt/libxml2/lib'
)
```

Please note that it is possible to pass argument variables within the command
line to instrument the build phase. As an example, the following is a possible
workflow to configure and build a customized 10.5 instance:

``` 
$ pgenv config write 10.5
pgenv config edit 10.5
# adjust PGENV_CONFIGURE_OPTIONS

$ pgenv build 10.5
```

In the case you need to specify a particular variable, such as the Perl
interpreter, pass it on the command line at the time of build:

``` 
PERL=/usr/local/my-fancy-perl pgenv build 10.5
```

At the end of a `build` (or a `rebuild`) phase, `pgenv` creates a configuration
file for the specific PostgreSQL version. If the file already exists, due to
a prior `build` or `rebuild` action, the file will be automatically overwritten.
In order to avoid the creation or overwriting of the configuration file,
it is possible to set the environment variable
`PGENV_WRITE_CONFIGURATION_FILE_AUTOMATICALLY` to a false value
(either `0` or `NO`). If the variable is not set at all,
or it is set to a true value (e.g., `1`, `YES`, etc.)
the configuration file will be created or  overwritten (if it already exists).

#### Patching

`pgenv` can patch the source tree before the build process starts. In
particular, the `patch/` folder can contain a set of index files and patch to
apply. The process searches for an index file corresponding to the PostgreSQL
version to build, and if found applies all the patches contained into the index.

Index files are named after the PostgreSQL version and the Operating System; in
particular the name of an index file is composed as `patch.<version>.<os>` where
`version` is the PostgreSQL version number (or a part of it) and `os` is the
Operating System. As an example, `patch.8.1.4.Linux` represents the index used
when building PostgreSQL 8.1.4 on a Linux machine. To provide more flexibility,
the system searches for an index that is named after the exact PostgreSQL
version and Operating System or a mix of possible combinations of those. As an
example, in the case of the PostgreSQL version 8.1.4 the index files is searched
among one of the following:

``` 
$PGENV_ROOT/patch/index/patch.8.1.4.Linux
$PGENV_ROOT/patch/index/patch.8.1.4
$PGENV_ROOT/patch/index/patch.8.1.Linux
$PGENV_ROOT/patch/index/patch.8.1
$PGENV_ROOT/patch/index/patch.8.Linux
$PGENV_ROOT/patch/index/patch.8
```

This allows you to specify an index for pretty much any combination or grouping
desired. The first index file that matches wins and it is the only one used for
the build process. If no index file is found at all, no patching is applied on
the source tree.

The index file must contain a list of patches to apply, that is file names
(either absolute or relative to the `patch/` subfolder). Each individual file is
applied thru `patch(1)`.

It is possible to specify a particular index file, that means avoid the
automatic index selection, by either setting the `PGENV_PATCH_INDEX` variable on
the command line or in the configuration file. As an example

``` 
$ PGENV_PATCH_INDEX=/src/my-patch-list.txt pgenv build 10.5
```

#### Build special keywords

The `build` command accepts the special keywords `earliest` and `latest` as 
indicators of the version to build. The logic is as follows:

-   `latest` triggers the build of the very last available PostgreSQL version
    available for download;
-   `earliest` triggers the build of the very first available PostgreSQL
    version, that is `1.08` (and probably is not what you are looking for);
-   `latest xx` where `xx` is a PostgreSQL major version number (e.g., `13`)
    triggers the build of the latest available version in such major version
    branch (e.g., `13.1`);
-   `earliest xx` where `xx` is a PostgreSQL major version number (e.g., `13`)
    triggers the build of the earliest available version in such major version
    branch (e.g., `13.0`).

The `latest` and `earliest` keywords work only with the `build` command and not
with the `rebuild` command.

### pgenv rebuild

The `rebuild` command allows the rebuilding from sources of a specific
PostgreSQL version. *The `PGDATA` directory will not be deleted if already
initialized via `initdb`*. However, in the case the PostgreSQL instance to
rebuild is currently in use, the `rebuild` command will not proceed. This is
meant to prevent the user to change the binaries of a in-use PostgreSQL cluster.

The configuration will follow the same rules adopted in `build`, which means if
a configuration file is present it will be loaded, otherwise the system will
claim about such file proposing to create one.

In the case a specific version has never been built, `rebuild` acts exactly as
`build`.

### pgenv remove

Removes the specified version of PostgreSQL unless it is the currently-active
version. Use the `clear` command to clear the active version before removing it.

```
$ pgenv remove 10.3
PostgreSQL 10.3 removed
```

The command removes the version, data directory, source code and configuration.

The `remove` command supports the special keywords `earliest` and `latest` to
respectively indicate the oldest PostgreSQL version installed and the newest
one. It is also possible to indicate a major version to narrow the scope of the
special keywords. As an example:

``` sh
pgenv remove latest 10
```
    
will remove the most recent PostgreSQL version of the 10 series installed.    

### pgenv start

Starts the currently active version of PostgreSQL if it's not already running.
Initializes the data directory if none exists.

``` 
$ pgenv start
PostgreSQL started
```
    
It is possible to specify flags to pass to `pg_ctl(1)` when performing the
`START` action, setting the `PGENV_START_OPTIONS` array in the
[configuration](#pgenv-config). Such options must not include the data
directory, nor the log file.

### pgenv stop

Stops the currently active version of PostgreSQL.

``` 
$ pgenv stop
PostgreSQL 10.5 stopped
```

It is possible to specify flags to pass to `pg_ctl(1)` when performing the
`stop` action, setting the `PGENV_STOP_OPTIONS` array in the
[configuration](#pgenv-config).

### pgenv restart

Restarts the currently active version of PostgreSQL, or starts it if it's not
already running.

``` 
$ pgenv restart
PostgreSQL 10.1 restarted
Logging to pgsql/data/server.log
```

It is possible to specify flags to pass to `pg_ctl(1)` when performing the
`restart` action, setting the `PGENV_RESTART_OPTIONS` array in the
[configuration](#pgenv-config).

### pgenv status

Indicates whether an instance is already *running* or is *stopped*. In case an
instance is not currently in use, the script complains and exits immediately.
Furthermore, you can use `pgenv current` to see which version is in use.

```sh
$ pgenv status
```

The above command results in an output like the following:

```
server is running (PID: 81803)
/opt/pgsql-16.0/bin/postgres "-D" "/opt/pgsql/data"
```

Same result can be achieved by running `pg_ctl` as follows:

```sh
$ $PG_ROOT/bin/pg_ctl status -D $PG_DATA
```

Where `PG_ROOT` is the path to your PostgreSQL installation directory, and
`PG_DATA` is the path to your database directory.

### pgenv available

Shows all the versions of PostgreSQL available to download and build. Handy to
help you finding a version to pass to the `build` command. Note that the
`available` command produces copious output.

``` 
$ pgenv available

            Available PostgreSQL Versions
========================================================
                  ...
 
                    PostgreSQL 9.6
    ------------------------------------------------
    9.6.0   9.6.1   9.6.2   9.6.3   9.6.4   9.6.5
    9.6.6   9.6.7   9.6.8   9.6.9   9.6.10

                    PostgreSQL 10
    ------------------------------------------------
    10.0    10.1    10.2    10.3    10.4    10.5

                    PostgreSQL 11
    ------------------------------------------------
     11.0    11.1    11.2    11.3    11.4    11.5   
     11.6    11.7    11.8    11.9    11.10   11.11  
     11.12   11.13  

                     PostgreSQL 12
    ------------------------------------------------
     12.0    12.1    12.2    12.3    12.4    12.5   
     12.6    12.7    12.8   

                     PostgreSQL 13
    ------------------------------------------------
     13.0    13.1    13.2    13.3    13.4   

                     PostgreSQL 14
    ------------------------------------------------
     14beta1  14beta2  14beta3  14rc1   14.0   

     
```

The versions are organized and sorted by major release number. Any listed
version may be passed to the `build` command.

To limit the list to versions for specific major releases, pass them to
`available`. For example, to list only the `9.6` and `10` available versions:

```  
$ pgenv available 10 9.6
            Available PostgreSQL Versions
========================================================

                    PostgreSQL 9.6
    ------------------------------------------------
    9.6.0   9.6.1   9.6.2   9.6.3   9.6.4   9.6.5
    9.6.6   9.6.7   9.6.8   9.6.9   9.6.10

                    PostgreSQL 10
    ------------------------------------------------
    10.0    10.1    10.2    10.3    10.4    10.5
```

### pgenv check

Checks the list of commands required to download and build PostgreSQL. Prints a
result for each, with either the path to the command or an error reporting that
the command was not found.

### pgenv help

Outputs a brief usage statement and summary of available commands, like the
following:

``` 
$ pgenv help
Usage: pgenv <command> [<args>]

The pgenv commands are:
    use        Set and start the current PostgreSQL version
    start      Start the current PostgreSQL server
    stop       Stop the current PostgreSQL server
    restart    Restart the current PostgreSQL server
    status     Show the current PostgreSQL server status
    switch     Set the current PostgreSQL version
    clear      Stop and unset the current PostgreSQL version
    build      Build a specific version of PostgreSQL
    rebuild    Re-build a specific version of PostgreSQL
    remove     Remove a specific version of PostgreSQL
    version    Show the current PostgreSQL version
    current    Same as 'version'
    versions   List all PostgreSQL versions available to pgenv
    help       Show this usage statement and command summary
    available  Show which versions can be downloaded
    check      Check all program dependencies
    config     View, edit, delete the program configuration
    log        Inspects the log of the cluster, if exist.

For full documentation, see: https://github.com/theory/pgenv#readme

This is 'pgenv' version 1.0.0 [5839e72]
```

The last line of the 'help' shows the `pgenv` version number and, if `git` is
available, the short commit hash (this can be useful when reporting bugs and
filling issue requests). Please note that, in order to print out the git `HEAD`
information, the `pgenv` must be able to find a `git` executable (i.e., it must
be in your `PATH`) and the `PGENV_ROOT` must be a git *checkout* directory.

### pgenv config

View, set, and delete configuration variables, both globally or for specific
versions of PostgreSQL. Stores the configuration in Bash files, one for each
version, as well as a default configuration. If pgenv cannot find a
configuration variable in a version-specific configuration file, it will look in
the default configuration. If it doesn't find it there, it tries to guess the
appropriate values, or falls back on its own defaults.

The `config` command accepts the following subcommands:


-    `show` prints the current or specified version configuration
-    `init` produces a configuration file from scratch, with default settings
-    `write` store the specified version configuration
-    `edit` opens the current or specified version configuration file in your favourite text editor 
           (Using `$EDITOR`, e.g: `export EDITOR=/usr/bin/emacs`)
-    `delete` removes the specified configuration
-    `migrate` is a command used to change the configuration format between versions of `pgenv`
-    `path` accepts a version number and prints on standard output the path to such version
            configuration path
 

Each sub-command accepts a PostgreSQL version number (e.g., `10.5`) or a
special keyword:

-   `current` or `version` tells pgenv to use the currently active version of
    PostgreSQL
-   `default` tells pgenv to use the default configuration;
-   `earliest` and `latest` to indicate respectively the oldest or newest
    version of PostgreSQL installed. As in other commands, these two keywords
    can be combined with a PostgreSQL major version number to point to the
    configuration of the earliest/latest version within that major number.


If no version is explicitly passed to any of the `config` subcommands, the
program will work against the currently active version of PostgreSQL.

In order to start with a default configuration, use the `write` subcommand:

``` 
$ pgenv config write default
pgenv configuration file ~/.pgenv/config/default.conf written
```

A subsequent `show` displays the defaults:

``` 
$ pgenv config show default
# Default configuration
# pgenv configuration for PostgreSQL
# File: /home/luca/git/misc/PostgreSQL/pgenv/config/default.conf
# ---------------------------------------------------
# pgenv configuration created on mer 12 set 2018, 08.35.52, CEST

# Enables debug output
# PGENV_DEBUG=''

###### Build settings #####
# Make command to use for build
# PGENV_MAKE=''

# Make flags
PGENV_MAKE_OPTIONS=(-j3)

# Configure flags
# PGENV_CONFIGURE_OPTIONS=( )
# ...

##### Runtime options #####
# Path to the cluster log file
PGENV_LOG='/home/luca/git/misc/PostgreSQL/pgenv/pgsql/data/server.log'

# Cluster administrator user
PGENV_PG_USER='postgres'

# Initdb flags
PGENV_INITDB_OPTIONS=(-U postgres --locale en_US.UTF-8 --encoding UNICODE)

# ...
```

You can edit the file and adjust parameters to your needs.


In order to create a configuration file for a specific version, it is possible
to use the `init` or `write` commands. If no prior default configuration exists,
the commands do the same, that is they create a from-scratch configuration file.
If a default configuration exists, the `write` command will "clone" such configuration
in a PostgreSQL version specific configuration file, while `init` will 
create a configuration file with default settings.
After a configuration file has been created by `init`, the `write` or `edit`
commands must be used against it, that means an existing configuration file
cannot be `init`ed more than once.

``` 
$ pgenv config write 10.5
pgenv configuration file [~/.pgenv/config/10.5.conf] written
```


Each time pgenv writes a configuration file, it first creates a backup with
the suffix `.backup` and a timestamp string related to when the backup file has
been created.

Use the `edit` subcommand to edit a configuration file in your favorite editor:

``` sh 
pgenv config edit 10.5
```

The `edit` command will start your favorite editor, that is whatever it is set
within the `EDITOR` variable. If such variable is not set you will be warned.
Use the `delete` subcommand to delete a configuration:

``` 
$ pgenv config delete 10.5
Configuration file ~/.pgenv/config/10.5.conf (and backup) deleted
```
   

The `delete` subcommand will not attempt to delete the default configuration
file, since it can be shared among different PostgreSQL versions.
However, if it is explicitly specified `default` as the version to delete
(i.e., `config delete default`), the default configuration file will be deleted.
      
``` 
$ pgenv config delete
Cannot delete default configuration while version configurations exist
To remove it anyway, delete ~/.pgenv/config/default.conf.
```
The `delete` subcommand deletes both the configuration file and its backup copy.
The `pgenv remove` command also deletes any configuration for the removed
version.

Please note that since commit [5839e721] the file name of the default
configuration file has changed. In the case you want to convert your default
configuration file, please issue a rename like the following

``` sh 
cp .pgenv.conf .pgenv.default.conf
```


The `migrate` command allows `pgenv` to change the configuration format of
the files between different releases. For example, it must be run if
you are upgrading `pgenv` from a version before `1.2.1` [811ba05], that changed the
location of configuration files into the `config` subdirectory.


```
pgenv config migrate
Migrated 3 configuration file(s) from previous versions (0 not migrated)
Your configuration file(s) are now into [~/git/misc/PostgreSQL/pgenv/config]
```


The `path` command accepts a specific version that will be used to compute the
configuration file name related to such version, printing out the resulting
path to the configuration file.
This allows the user to set the `PGENV_CONFIGURATION_FILE` environment variable
to a specific path to a custom configuration file, so that other subsequent
invocations of `pgenv` will refer to such path.
As an example, this is a way to exploit the default configuration file
for different versions of PostgreSQL.
In order to export the variable to a specific custom file location
you can do, in your terminal, something like the following:

```
export PGENV_CONFIGURATION_FILE=$( pgenv config path 15.4 )
```

The above will set the environment variable `PGENV_CONFIGURATION_FILE` to the configuration
file for the PostgreSQL version `15.4`.
If you want to use the default configuration file, substitute the version number with the
special `default` keyowrd, for example:

```
export PGENV_CONFIGURATION_FILE=pgenv config path default
```



### pgenv log

The `log` command provides a dump of the cluster log, if it exists, so that you
don't have to worry about the exact log location. The log is dumped using the
`tail` command, and every option passed to the command line is passed thru
`tail`. As an example:

``` 
$ pgenv log     
Dumping the content of /home/luca/git/misc/PostgreSQL/pgenv/pgsql/data/server.log 

LOG:  could not bind IPv4 address "127.0.0.1": Address already in use
HINT:  Is another postmaster already running on port 5432? If not, wait a few seconds and retry.
WARNING:  could not create listen socket for "localhost"
FATAL:  could not create any TCP/IP sockets
LOG:  database system is shut down
LOG:  starting PostgreSQL 12.1 on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 8.3.0-6ubuntu1) 8.3.0, 64-bit
LOG:  listening on IPv4 address "127.0.0.1", port 5432
LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
LOG:  database system was shut down at 2020-08-28 12:57:33 CEST
LOG:  database system is ready to accept connections
 ```

The above is equivalent to manually executing

``` sh
tail ~/git/misc/PostgreSQL/pgenv/pgsql/data/server.log
```

It is possible to pass arguments to `tail` as command line flags:

``` 
$ pgenv log -n 2
Dumping the content of /home/luca/git/misc/PostgreSQL/pgenv/pgsql/data/server.log 

LOG:  database system was shut down at 2020-08-28 12:57:33 CEST
LOG:  database system is ready to accept connections
```

which results in executing

``` sh
tail -n 2 /home/luca/git/misc/PostgreSQL/pgenv/pgsql/data/server.log 
```

and of course, you can inspect the log of live system continuously:

``` sh
pgenv log -f
```    

# Bug Reporting

Please use [GitHub issues].

See Also
--------

*   [plenv] is a binary manager for [Perl], and was the inspiration for pgenv.
*   plenv, in turn, was inspired by and based on [rbenv], a binary manager for
    Ruby.
*   [Pgenv] works similarly, but requires [PostgreSQL] manually compiled from
    its Git repo.

License
-------

Distributed under [The MIT License]; see [`LICENSE.md`] for terms.

[PostgreSQL]: https://postgresql.org/
[plenv]: https://github.com/tokuhirom/plenv/
[Perl]: https://perl.org/
[rbenv]: https://github.com/rbenv/rbenv
[Pgenv]: https://github.com/mnencia/pgenv
[GitHub issues]: https://github.com/theory/pgenv/issues/
[The MIT License]: https://opensource.org/licenses/MIT
[`LICENSE.md`]: https://github.com/theory/pgenv/blob/master/LICENSE.md
[5839e721]: https://github.com/theory/pgenv/commit/5839e721d43c9eae8b4a0d61ba78996c220a4da0
