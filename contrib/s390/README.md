# How to enable IBM LinuxONE III zEDC Integrated Accelerator

## Decompression

Decompression is always hardware-accelerated by default.

## Compression

All the software (e.g. Java), when using zlib, internally chooses a tradeoff between compression performance and
quality. This concept is known as compression level - a number between `1` and `9`, where `1` stands for best
performance and `9` stands for best quality.

The hardware accelerator has no notion of compression level - the compression quality it delivers is always roughly the
same as software compression level `1`. This is why by default it is enabled only on level `1`.

If the software you are using allows you to choose the compression level, choose `1` in order to enable the hardware
acceleration.

**Use the following steps only if the compression level other than `1` is hardcoded in your software.**

## Environment variables

Whether hardware acceleration is used, is controlled by two environment variables:

* `DFLTCC`: `1` (default) means hardware acceleration is on, `0` means hardware acceleration is off.
* `DFLTCC_LEVEL_MASK`: bit mask, which selects compression levels to be accelerated. Default is `2`.

In order to enable the hardware acceleration in software, which does not allow the user to control the compression
level, you need to tweak `DFLTCC_LEVEL_MASK`. A good catch-all value is `0x1fe`, which corresponds to levels 1-8. Some
software might have level 9 hardcoded, in which case a value of `0x2fe` (levels 1-9) will be required.

### Single command

If all you need is running a single command with the hardware acceleration, use [`env`](
http://man7.org/linux/man-pages/man1/env.1.html):

`env DFLTCC_LEVEL_MASK=0x2fe COMMAND [ARGS]`

It's ok to use a very aggressive `0x2fe` value here, since it will affect only a single command.

### All users

Use the [global `/etc/environment`](https://www.freedesktop.org/software/systemd/man/environment.d.html):

```
echo DFLTCC_LEVEL_MASK=0x1fe >>/etc/environment
```

It is better to use the conservative `0x1fe` here. Use `0x2fe` at your own risk!

### Specific user

Unfortunately, there is no universal way to do this. Assuming you need this only for interactive bash sessions:

```
echo DFLTCC_LEVEL_MASK=0x1fe >>~/.bashrc
```

### All systemd services

Use the [global systemd config file](https://www.freedesktop.org/software/systemd/man/systemd-system.conf.html):

```
printf "[Manager]\nDefaultEnvironment=DFLTCC_LEVEL_MASK=0x1fe\n" >/etc/systemd/system.conf.d/dfltcc.conf
```

### Specific systemd service

Use the [systemd unit override](https://www.freedesktop.org/software/systemd/man/systemd.unit.html):

```
printf "[Service]\nEnvironment=DFLTCC_LEVEL_MASK=0x2fe\n" >/etc/systemd/system/YOUR-SERVICE.service.d/dfltcc.conf
```

### Docker container

Use [`-e`](https://docs.docker.com/engine/reference/run/#env-environment-variables):

`docker run -it --rm -e DFLTCC_LEVEL_MASK=0x2fe IMAGE COMMAND [ARGS]`

Make sure zlib in your image supports hardware acceleration for this to work.

# See also

* [zlib acceleration](https://linux.mainframe.blog/zlib-acceleration/)
* [http://linux-on-z.blogspot.com/2019/10/howto-exploiting-hardware-compression.html](
http://linux-on-z.blogspot.com/2019/10/howto-exploiting-hardware-compression.html)
