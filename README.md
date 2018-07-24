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
   
    # Show current version
    pgenv version
    
    # List built versions
    pgenv versions

Description
-----------

pgenv is a simple utility to build and run different releases of [PostgreSQL].
This makes it easy to switch between versions when testing for compatability.

Installation
------------

1.  Check out plenv into `~/.plenv`.

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
    $ pgenv install 10.4
    ~~~

See Also
--------

*   [plenv] is a binary manager for [Perl], and was the inspiration for pgenv.
*   [Pgenv] works similarly, but requires [PostgreSQL] manually compiled from
    its Git repo.

[PostgreSQL]: https://postgresql.org/
[plenv]: https://github.com/tokuhirom/plenv/
[Perl]: https://perl.org/
[Pgenv]: https://github.com/mnencia/pgenv
