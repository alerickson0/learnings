# Objective
For a personal project, I was trying to run a local version of Redis Open Source on my WSL Rocky 10 Linux. I hope any info or guidance in here is useful. Please note, this doc is currently in draft status. Because I'm still working on it.

# Setup
Currently, as of November 2025, there is no install or package one can get of the open source version of Redis.
So, you have to compile it from source yourself.

Now, here was my setup:
- Main OS: Windows 11 25H2
- WSL: Rocky Linux 10.0 (Red Quartz)
- WSL Python version being used via a Python virtual environment: 3.14.0

# Steps I followed to compile Redis Open Source
Like many folks, I tried following one guide[^1] only to run into issues. After some internet searching I found other guides and answers (links to the original guides I followed can be found in the References section). Then, trying some commands and such, I was able to compile redis. So, here are the steps together:
1. Enable Python virtual environment you want to use
2. Download the Redis source code as a tarball:
    1. `redisurl="http://download.redis.io/redis-stable.tar.gz"`
    2. `curl -s -o redis-stable.tar.gz $redisurl`
3. Next, switch over to the *root* user and extract the archive's source code to `/usr/local/bin`:
    1. `sudo su root`
    2. `mkdir -p /usr/local/lib/`
    3. `chmod a+w /usr/local/lib/`
    4. `tar -C /usr/local/lib/ -xzf redis-stable.tar.gz`
4. Now, there will be a local source code repo at `/usr/local/lib/redis-stable/`. So, go into it:
    1. `cd /usr/local/lib/redis-stable/`
5. I found I needed to install to install `make` to do the compile step. And I did it by doing the following[^2]:
    1. `dnf makecache --refresh`
    2. `dnf -y install make`
6. Found I need to install additional tools and packages as listed here[^3]. Enable the GoReleaser repo:
```
    tee /etc/yum.repos.d/goreleaser.repo > /dev/null <<EOF
    [goreleaser]
    name=GoReleaser
    baseurl=https://repo.goreleaser.com/yum/
    enabled=1
    gpgcheck=0
    EOF

    dnf clean all
    dnf makecache
    dnf update -y
```

7. Next, the tools and packages I installed were only ones that weren't present on my OS:

```
  dnf install -y --nobest --skip-broken \
    xz \
    wget \
    openssl \
    openssl-devel \
    clang \
    libtool \
    automake \
    autoconf \
    jq \
    systemd-devel
```
8. Install `cmake`, now I'm unsure if this is needed. But it doesn't hurt to have:
    1. `dnf install cmake`
9. Finally, run the `make` commands to do the compile:
    1. `make && make install`

Finally, you will be able to see the redis binaries under `/usr/local/bin/`.


# References
[^1]: [Installing Redis From Source](https://realpython.com/python-redis/#installing-redis-from-source)

[^2]: Commands still work for Rocky v10: [How To Install make on Rocky Linux 8](https://installati.one/install-make-rockylinux-8/#install-make-on-rocky-linux-8-using-dnf)

[^3]: From the official Redis docs for installing/compiling Redis on Rocky v9.5: [Install required packages - original source](https://redis.io/docs/latest/operate/oss_and_stack/install/build-stack/almalinux-rocky-9/#2-install-required-packages)
