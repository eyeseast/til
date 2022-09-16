# Installing a custom SQLite version on Ubuntu (in docker)

I have a docker image running a [Datasette](https://www.datasette.io) instance. I wanted to use the latest version of SQLite, which is [3.39.3](https://www.sqlite.org/releaselog/3_39_3.html) as of September 2022. Running `apt install sqlite3` on Ubuntu 22.04 gets me [3.37.2](https://www.sqlite.org/releaselog/3_37_2.html), released in January. I really wanted to use the new JSON query syntax that landed a few versions later.

This presents two problems:

1. How to install the latest version
2. How to get Python to use it, instead of the system version

## Compile SQLite

Ultimately, the way to get the most recent version is to pull down the source code and compile it myself. Thankfully, SQLite makes this straightforward:

```sh
SQLITE_VERSION=3390300
wget -O sqlite.tar.gz https://www.sqlite.org/2022/sqlite-autoconf-${SQLITE_VERSION}.tar.gz
tar -xzvf sqlite.tar.gz
cd sqlite-autoconf-${SQLITE_VERSION}
./configure
make
make install
```

Download, unpack, `./configure`, `make` and `make install`. There's a lot more detail [here](https://www.sqlite.org/howtocompile.html) and lots of compile-time options. All the defaults were fine for me.

## Get Python to use the new version

I thought I'd have to compile Python, too, but it turns out it can load SQLite dynamically by setting the `LD_PRELOAD` environment variable. [Simon Willison has details here](https://til.simonwillison.net/sqlite/ld-preload). And [here](https://www.baeldung.com/linux/ld_preload-trick-what-is) is a good explainer on how `LD_PRELOAD` works.

Pointing `LD_PRELOAD` at the compiled SQLite library, Python will use that instead of the default version. In my case, this worked:

```sh
LD_PRELOAD=/usr/local/lib/libsqlite3.so
```

Here's the relevant part of the Dockerfile:

```dockerfile
FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive \
    SQLITE_VERSION=3390300 \
    LD_PRELOAD=/usr/local/lib/libsqlite3.so

RUN apt-get update && \
    apt-get -y --no-install-recommends install \
    python3-dev python3-pip cmake build-essential tcl wget && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN wget -O sqlite.tar.gz https://www.sqlite.org/2022/sqlite-autoconf-${SQLITE_VERSION}.tar.gz && \
    tar -xzvf sqlite.tar.gz && \
    (cd sqlite-autoconf-${SQLITE_VERSION} && ./configure && make && make install)
```

When it's time to upgrade, I can change the `SQLITE_VERSION` variable and rebuild.
