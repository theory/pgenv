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
    
    # Use PostgreSQL version
    pgenv use 10.4
    
    # Stop current version
    pgenv stop
    
    # Start current version
    pgenv start

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
    $ git clone https://github.com/theory/pgenv.git ~/.pgenv
    ```

2.  Add `~/.pgenv/bin` and `~/.pgenv/pgsql/bin` to your `$PATH` for access to
    the `pgenv` command-line utility and all the programs provided by
    PostgreSQL:
   
    ``` sh
    $ echo 'export PATH="$HOME/.pgenv/bin:$HOME/.pgenv/pgsql/bin:$PATH"' >> ~/.bash_profile
    ```

    **Ubuntu note**: Modify your `~/.profile` instead of `~/.bash_profile`.

    **Zsh note**: Modify your `~/.zshrc` file instead of `~/.bash_profile`.

3.  Restart your shell as a login shell so the path changes take effect. You
    can now begin using pgenv.

    ~~~ sh
    $ exec $SHELL -l
    ~~~

4.  Build a version of PostgreSQL:

    ~~~ sh
    $ pgenv build 10.4
    ~~~

### Configuration

By default, all versions of PostgreSQL will be built in the root of the
project directory (generally in `~/.pgenv`.). If you'd like them to live
elsewhere, set the `$PGENV_ROOT` environment variable to the appropriate
directory.

It is possible to configure which programs, flags and languages to build
using a configuration file before the program launches the build. 
For a more detailed configuration, see the [`pgenv config`](#pgenv-config)
command below.

### Upgrading

You can upgrade your installation to the cutting-edge version at any time
with a simple `git pull`.

~~~ sh
$ cd ~/.pgenv
$ git pull
~~~

### Dependencies
------------

*   env - Sets environment for execution
*   bash - Command shell interpreter
*   curl - Used to download files
*   sed, grep, cat, tar, sort, tr, uname - General Unix command line utilities
*   patch - For patching versions that need patching
*   make -  Builds PostgreSQL

Optional dependencies:

*   Perl 5 - To build PL/Perl 
*   Python - To build PL/Python

Command Reference
-----------------

Like `git`, the `pgenv` command delegates to subcommands based on its
first argument. The subcommands are:

### pgenv use

Sets the version of PostgreSQL to be used in all shells by symlinking its
directory to `~/$PGENV_ROOT/pgsql` and starting it. Initializes the data
directory if none exists. If another version is currently active, it will be
stopped before switching. If the specified version is already in use, the
`use` command won't stop it, but will initialize its data directory and starts
it if it's not already running.

    $ pgenv use 10.4
    waiting for server to shut down.... done
    server stopped
    waiting for server to start.... done
    server started
    PostgreSQL 10.4 started

### pgenv versions

Lists all PostgreSQL versions known to `pgenv`, and shows an asterisk next to
the currently active version (if any). The first column reports versions
available for use by `pgenv` and the second lists the subdirectory of
`$PGENV_ROOT` in which the each version is installed:

    $ pgenv versions
          10.4      pgsql-10.4
          11beta3   pgsql-11beta3
          9.5.13    pgsql-9.5.13
      *   9.6.9     pgsql-9.6.9

In this example, versions `9.5.13`, `9.6.9`, `10.4`, and `11beta3` are
available for use, and the `*` indicates that `9.6.10` is the currently active
version. Each version is installed in a `pgsql-` subdirectory of
`$PGENV_ROOT`.

### pgenv current

Displays the currently active PostgreSQL version.

    $ pgenv current
    10.4

Please note that `version` is a command synonym for version:

    $ pgenv version
    10.4

### pgenv clear

Clears the currently active version of PostgreSQL. If the current version is
running, `clear` will stop it before clearing it.

    $ pgenv clear
    waiting for server to shut down.... done
    server stopped
    PostgreSQL stopped
    PostgreSQL cleared

### pgenv build

Downloads and builds the specified version of PostgreSQL and its contrib
modules, as far back as 8.0. 
It is possible to instrument the build process to patch the source tree, see
the section on patching later on.
If the version is already built, it will not be rebuilt; use
`clear` to remove an existing version before building it again.

    $ pgenv build 10.3
    # [Curl, configure, and make output elided]
    PostgreSQL 10.3 built

The build phase can be customized via a configuration file, in the case
the system does not find a configuration file when the `build` is executed,
a warning is shown to the user to remind she can edit a configuration file
and start over the build process:

```sh
$ pgenv build 10.3
  ...
WARNING: no configuration file found for version 10.3
HINT: if you wish to customize the build process please
stop the execution within 5 seconds (CTRL-c) and run
    pgenv config write 10.3 && pgenv config edit 10.3
adjust 'configure' and 'make' options and flags and run again
    pgenv build 10.3
```

Within the configuration file it is possible to instrument
the build phase from the configuration to the actual build. For instance,
in order to build with PL/Perl, it is possible to configure
the variable `PGENV_CONFIGURE_OPTS` adding `--with-perl`. Please note that
it is possible to pass argument variables within the command line to
instrument the build phase. As an example, the following is a possible
work-flow to configure and build a customized 10.5 instance:


    $ pgenv config write 10.5
    $ pgenv config edit 10.5
    # adjust PGENV_CONFIGURE_OPTS
    $ pgenv build 10.5

In the case you need to specify a particular variable, such as the Perl interpreter,
pass it on the command line at the time of build:

   $ PERL=/usr/local/my-fancy-perl pgenv build 10.5
   
#### Patching

`pgenv` can patch the source tree before the build process starts.
In particular, the `patch/` folder can contain a set of index files
and patch to apply. The process searches for an index file corresponding
to the PostgreSQL version to build, and if found applies all the patches
contained into the index.

Index files are named after the PostgreSQL version and the Operating System;
in particular the name of an index file is composed as `patch.<version>.<os>`
where `version` is the PostgreSQL version number (or a part of it) and
`os` is the Operating System. As an example, `patch.8.1.4.Linux` represents
the index used when building PostgreSQL 8.1.4 on a Linux machine.
To provide more flexibility, the system searches for an index that is named
after the exact PostgreSQL version and Operating System or a mix of
possible combinations of those.
As an example, in the case of the PostgreSQL version
8.1.4 the index files is searched among one of the following:

      $PGENV_ROOT/patch/index/patch.8.1.4.Linux
      $PGENV_ROOT/patch/index/patch.8.1.4
      $PGENV_ROOT/patch/index/patch.8.1.Linux
      $PGENV_ROOT/patch/index/patch.8.1
      $PGENV_ROOT/patch/index/patch.8.Linux
      $PGENV_ROOT/patch/index/patch.8


This allows you to specify an index for pretty much any combination or grouping desired.
The first index file that matches wins and it is the only one used for the build process. 
If no index file is found at all, no patching is applied on the source tree.

The index file must contain a list of patches to apply, that is file names (either absolute
or relative to the `patch/` subfolder). Each individual file is applied thru `patch(1)`.

It is possible to specify a particular index file, that means avoid the automatic index selection,
by either setting the `PGENV_PATCH_INDEX` variable on the command line or in the configuration
file. As an example

    $ PGENV_PATCH_INDEX=/src/my-patch-list.txt pgenv build 10.5



### pgenv remove

Removes the specified version of PostgreSQL unless it is the currently-active
version. Use the `clear` command to clear the active version before removing it.

    $ pgenv remove 10.3
    PostgreSQL 10.3 removed

The command removes the version, data directory, source code and configuration.

### pgenv start

Starts the currently active version of PostgreSQL if it's not already running.
Initializes the data directory if none exists.

    $ pgenv start
    PostgreSQL started
    
It is possible to specify flags to pass to `pg_ctl(1)` when performing the
`START` action, setting the `PGENV_START_OPTS` in the [configuration](#pgenv-config).
Such options must not include the data directory, nor the log file.

### pgenv stop

Stops the currently active version of PostgreSQL.

    $ pgenv stop
    PostgreSQL 10.5 stopped

It is possible to specify flags to pass to `pg_ctl(1)` when performing the
`stop` action, setting the `PGENV_STOP_OPTS` in the [configuration](#pgenv-config).

### pgenv restart

Restarts the currently active version of PostgreSQL, or starts it if it's not
already running.

    $ pgenv restart
    PostgreSQL 10.1 restarted
    Logging to pgsql/data/server.log

It is possible to specify flags to pass to `pg_ctl(1)` when performing the
`restart` action, setting the `PGENV_RESTART_OPTS` in the [configuration](#pgenv-config).


### pgenv available

Shows all the versions of PostgreSQL available to download and build. Handy to
find a version you to pass to the `build` command. Note that the `available`
command produces copious output.

    $ pgenv available
    ...
                Available PostgreSQL Versions
    ========================================================

                        PostgreSQL 9.6
        ------------------------------------------------
        9.6.0   9.6.1   9.6.2   9.6.3   9.6.4   9.6.5
        9.6.6   9.6.7   9.6.8   9.6.9   9.6.10

                        PostgreSQL 10
        ------------------------------------------------
        10.0    10.1    10.2    10.3    10.4    10.5

                        PostgreSQL 11
        ------------------------------------------------
        11beta1  11beta2  11beta3

The versions are organized and sorted by major release number. Any listed
version may be passed to the `build` command.

To limit the list to versions for specific major releases, pass them to
`available`. For example, to list only the `9.6` and `10` available versions:

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

### pgenv check

Checks the list of commands required to download and build PostgreSQL. Prints
a result for each, with either the path to the command or an error reporting
that the command was not found.

### pgenv help

Outputs a brief usage statement and summary of available commands, like
the following:

    $ pgenv help
    Usage: pgenv <command> [<args>]
    The pgenv commands are:
        use        Set and start the current PostgreSQL version
        clear      Stop and unset the current PostgreSQL version
        start      Start the current PostgreSQL server
        stop       Stop the current PostgreSQL server
        restart    Restart the current PostgreSQL server
        build      Build a specific version of PostgreSQL
        remove     Remove a specific version of PostgreSQL
        version    Show the current PostgreSQL version
        current    Same as 'version'
        versions   List all PostgreSQL versions available to pgenv
        help       Show this usage statement and command summary
        available  Show which versions can be downloaded
        check      Check all program dependencies
        config     View, edit, delete the program configuration

    For full documentation, see: https://github.com/theory/pgenv#readme

### pgenv config

View, set, and delete configuration variables, both globally or for specific
versions of PostgreSQL. Stores the configuration in Bash files, one for each
version, as well as a default configuration. If pgenv cannot find a
configuration variable in a version-specific configuration file, it will look
in the default configuration. If it doesn't find it there, it tires to guess
the appropriate values, or falls back on its own defaults.

The `config` command accepts the following subcommands:

- `show` prints the current or specified version configuration
- `write` store the current or specified version configuration
- `edit` opens the current or specified version configuration in an editor (Using $EDITOR, e.g: export EDITOR=/usr/bin/vim)
- `delete` removes the specified configuration

Each sub-command accepts a PostgreSQL version number (e.g., `10.5`) or a
special keyword:

- `current` or `version` tells pgenv to use the currently active version of
  PostgreSQL
- `default` tells pgenv to use the default configuration.

If no version is explicitly passed to any of the `config` subcommands, the
program will work against the currently activce version of PostgreSQL.

In order to start with a default configuration, use the `write` subcommand:

    $ pgenv config write default
    pgenv configuration file ~/.pgenv/.pgenv.conf written

A subsequent `show` displays the defaults:

``` sh
$ pgenv config show default
# Default configuration
# pgenv configuration for PostgreSQL
# File: /home/luca/git/misc/PostgreSQL/pgenv/.pgenv.conf
# ---------------------------------------------------
# pgenv configuration created on mer 12 set 2018, 08.35.52, CEST

# Enables debug output
# PGENV_DEBUG=''

###### Build settings #####
# Make command to use for build
# PGENV_MAKE=''

# Make flags
PGENV_MAKE_OPTS='-j3'

# Configure flags
# PGENV_CONFIGURE_OPTS=''


# ...

##### Runtime options #####
# Path to the cluster log file
PGENV_LOG='/home/luca/git/misc/PostgreSQL/pgenv/pgsql/data/server.log'

# Cluster administrator user
PGENV_PG_USER='postgres'

# Initdb flags
PGENV_INITDB_OPTS='-U postgres --locale en_US.UTF-8 --encoding UNICODE'

# ...
```

You can edit the file and adjust parameters to your needs.

In order to create a configuration file for a specific version, pass the
version to the `write` subcommand:

   $ pgenv config write 10.5
   pgenv configuration file [~/.pgenv/.pgenv.10.5.conf] written

Each time pgenv writes a configuration file, it first creates a backup with
the suffix `.backup`.

Use the `edit` subcommand to edit a configuration file in your favourite
editor:

   $ pgenv config edit 10.5
   <output omitted>

The `edit` command will start your favourite editor, that is whatever
it is set within the `EDITOR` variable. If such variable is not set
you will be warned.
Use the `delete` subcommand To delete a configuration:

   $ pgenv config delete 10.5
   Configuration file ~/.pgenv/.pgenv.10.5.conf (and backup) deleted

Note that you cannot delete the default configuration unless all other
versions have been removed (e.g., by `pgenv remove`). This prevents accidental
loss of configuration:

   $ pgenv config delete
   Cannot delete default configuration while version configurations exist
   To remove it anyway, delete ~/.pgenv/.pgenv.conf.

The `delete` subcommand deletes both the configuration file and its backup
copy. The `pgenv remove` command also deletes any configuration for the
removed version.

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
