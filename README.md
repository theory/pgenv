pgenv - PostgreSQL binary manager
=================================

Synopsis
--------

    pgenv help

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
project directory (generally, in `~/.pgenv`.). If you'd like them to live
elsewhere, set the `$PGENV_ROOT` environment variable to the appropriate
value.

### Upgrading

You can upgrade your installation to the cutting-edge version at any time
with a simple `git pull`.

~~~ sh
$ cd ~/.pgenv
$ git pull
~~~

### Dependencies
------------

*   bash
*   curl - Used to download files
*   sed, grep, cat, tar - General Unix command line utilities
*   patch - For patching versions that need patching
*   make -  Builds PostgreSQL
*   Perl 5 - To build PL/Perl

Command Reference
-----------------

Like `git`, the `pgenv` command delegates to subcommands based on its
first argument. The subcommands are:

### pgenv use

Sets the version of PostgreSQL to be used in all shells by symlinking its
directory to `~/.pgenv/pgsql` and starting it. Initializes the data directory if
none exists. If another version is currently active, it will be stopped before
switching. If the specified version is already in use, the `use` command won't
stop it, but will initialize its data directory and start it if it is not
already running.

    $ pgenv use 10.4
    waiting for server to shut down.... done
    server stopped
    waiting for server to start.... done
    server started
    PostgreSQL 10.4 started

### pgenv versions

Lists all PostgreSQL versions known to `pgenv`, and shows an asterisk next to the
currently active version (if any). The first column reports the version number to use
in other `pgenv` commands, followed by the directory where the version is installed:

    $ pgenv versions
            10.5            pgsql-10.5
            10.3            pgsql-10.3
     *      9.6.10          pgsql-9.6.10

In the above output, version `9.6.10` is currently in use (note the '*'),
and available versions are `9.6.10`, `10.3` and `10.5` which are installed
in a `pgsql-` local directory named after the version number.

### pgenv version

Displays the currently active PostgreSQL version.

    $ pgenv version
    pgsql-10.4

### pgenv clear

Clears the currently active version of PostgreSQL. If the current version is running,
`clear` will stop it before clearing it.

    $ pgenv clear
    waiting for server to shut down.... done
    server stopped
    PostgreSQL stopped
    PostgreSQL cleared

### pgenv build

Downloads and builds the specified version of PostgreSQL and its contrib
modules, as far back as 8.0. PostgreSQL versions 8.0 and 8.1 will be patched
before building. If the version is already built, it will not be rebuilt; use
`clear` to remove an existing version before building it again.

    $ pgenv build 10.3
    # Curl, configure, and make output elided
    PostgreSQL 10.3 built

### pgenv remove

Removes the specified version of PostgreSQL unless it is the currently-active
version. Use the `clear` command to clear the active version before removing it.

    $ pgenv remove 10.3
    PostgreSQL 10.3 removed

### pgenv start

Starts the currently active version of PostgreSQL if it's not already running.
Initializes the data directory if none exists.

    $ pgenv start
    PostgreSQL started

### pgenv stop

Stops the currently active version of PostgreSQL.

    $ pgenv stop
    PostgreSQL stopped

### pgenv restart

Restarts the currently active version of PostgreSQL, or starts it if it's not
already running.

    $ pgenv restart
    PostgreSQL restarted

### pgenv available

Shows all the versions of PostgreSQL available to download and build. Handy to
find a version you want to pass to the `build` command. Note that this command
produces copious output.

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

The versions are organized by major release version. Any listed version may be
passed to `pgenv` commands that require a version.

To limit the list to specific major releases, pass those releases to
`available`. For example, to list only the `9.6` and `10` available versions:

    $ pgenv available 9.6 10
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

Checks the list of commands required to download and build PostgreSQL. Prints a
result for each, with either the path to the command or an error reporting that
the command was not found.

### pgenv help

Outputs a brief usage statement and summary of available commands, like
the following:

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
        versions   List all PostgreSQL versions available to pgenv
        help       Show this usage statement and command summary
        available  Show which versions can be downloaded (most recent versions last)
        check      Check all program dependencies

    For full documentation, see: https://github.com/theory/pgenv#readme

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
