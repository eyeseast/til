# How to load SQLite extensions (like spatialite) when using pyenv

I started using pyenv a few months ago. It's great. I recommend it. But I ran into this issue trying to use [spatialite](https://www.gaia-gis.it/fossil/libspatialite/index):

```
datasette census.db --load-extension spatialite
Traceback (most recent call last):
  File "/Users/camico/.local/bin/datasette", line 8, in <module>
    sys.exit(cli())
  File "/Users/camico/.local/pipx/venvs/datasette/lib/python3.9/site-packages/click/core.py", line 829, in __call__
    return self.main(*args, **kwargs)
  File "/Users/camico/.local/pipx/venvs/datasette/lib/python3.9/site-packages/click/core.py", line 782, in main
    rv = self.invoke(ctx)
  File "/Users/camico/.local/pipx/venvs/datasette/lib/python3.9/site-packages/click/core.py", line 1259, in invoke
    return _process_result(sub_ctx.command.invoke(sub_ctx))
  File "/Users/camico/.local/pipx/venvs/datasette/lib/python3.9/site-packages/click/core.py", line 1066, in invoke
    return ctx.invoke(self.callback, **ctx.params)
  File "/Users/camico/.local/pipx/venvs/datasette/lib/python3.9/site-packages/click/core.py", line 610, in invoke
    return callback(*args, **kwargs)
  File "/Users/camico/.local/pipx/venvs/datasette/lib/python3.9/site-packages/datasette/cli.py", line 544, in serve
    asyncio.get_event_loop().run_until_complete(check_databases(ds))
  File "/Users/camico/.pyenv/versions/3.9.2/lib/python3.9/asyncio/base_events.py", line 642, in run_until_complete
    return future.result()
  File "/Users/camico/.local/pipx/venvs/datasette/lib/python3.9/site-packages/datasette/cli.py", line 584, in check_databases
    await database.execute_fn(check_connection)
  File "/Users/camico/.local/pipx/venvs/datasette/lib/python3.9/site-packages/datasette/database.py", line 155, in execute_fn
    return await asyncio.get_event_loop().run_in_executor(
  File "/Users/camico/.pyenv/versions/3.9.2/lib/python3.9/concurrent/futures/thread.py", line 52, in run
    result = self.fn(*self.args, **self.kwargs)
  File "/Users/camico/.local/pipx/venvs/datasette/lib/python3.9/site-packages/datasette/database.py", line 151, in in_thread
    self.ds._prepare_connection(conn, self.name)
  File "/Users/camico/.local/pipx/venvs/datasette/lib/python3.9/site-packages/datasette/app.py", line 502, in _prepare_connection
    conn.enable_load_extension(True)
AttributeError: 'sqlite3.Connection' object has no attribute 'enable_load_extension'
```

It turns out pyenv, by default, [builds Python without a flag that enables SQLite extensions](https://github.com/pyenv/pyenv/issues/1702). Here's the fix:

```sh
# Do the following in your shell
LDFLAGS="-L/usr/local/opt/sqlite/lib -L/usr/local/opt/zlib/lib" CPPFLAGS="-I/usr/local/opt/sqlite/include -I/usr/local/opt/zlib/include" PYTHON_CONFIGURE_OPTS="--enable-loadable-sqlite-extensions" pyenv install 3.7.6
```

[Source](https://stackoverflow.com/questions/58892028/sqlite3-connection-object-has-no-attribute-enable-load-extension) for that fix. The `PYTHON_CONFIGURE_OPTS` variable tells pyenv to add the `--enable-loadable-sqlite-extensions` flag when compiling Python. The other variables tell it where to find various shared header libraries.
