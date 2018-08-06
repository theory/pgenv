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
*   curl
*   Perl

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

Lists all PostgreSQL versions known to pgenv, and shows an asterisk next to the
currently active version.

    $ pgenv versions
         pgsql-10.4
      *  pgsql-11beta2
         pgsql-8.0.26
         pgsql-8.1.23
         pgsql-8.2.23
         pgsql-8.3.23
         pgsql-8.4.22
         pgsql-9.0.19
         pgsql-9.1.24
         pgsql-9.2.24
         pgsql-9.3.23
         pgsql-9.4.18
         pgsql-9.5.13
         pgsql-9.6.9

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

### pgenv help

Outputs a brief usage statement and summary of available commands. The

    $ pgenv help
    Usage: pgenv <command> [<args>]"

    The pgenv commands are:
        use       Set and start the current PostgreSQL version
        clear     Stop and unset the current PostgreSQL version
        start     Start the current PostgreSQL server
        stop      Stop the current PostgreSQL server
        restart   Restart the current PostgreSQL server
        build     Build a specific version of PostgreSQL
        remove    Remove a specific version of PostgreSQL
        version   Show the current PostgreSQL version
        versions  List all Perl versions available to pgenv
        help      Show this usage statement and command summary
    
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
