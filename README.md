## MAVLink Router ##

[![Build Status](https://travis-ci.org/01org/mavlink-router.svg?branch=master)](https://travis-ci.org/01org/mavlink-router) [![Coverity Scan Build Status](https://scan.coverity.com/projects/11557/badge.svg)](https://scan.coverity.com/projects/11557)

Route mavlink packets between endpoints (WIP)

In its current state it acts as a bridge between one "master" endpoint on UART
and other endpoints on UDP.

### Compilation and installation ###

In order to compile you need the following packages:

  - GCC or Clang compiler
  - C and C++ standard libraries

#### Fetch dependencies: ####

We currently depend on mavlink C library which is generated by the build
system during compilation. The corresponding submodule should be fetched.

    $ git submodule update --init --recursive

#### Build ####

Build system follows the usual configure/build/install cycle. Configuration is needed
to be done only once. A typical configuration is shown below:

    $ ./autogen.sh && ./configure CFLAGS='-g -O2' \
            --sysconfdir=/etc --localstatedir=/var --libdir=/usr/lib64 \
	    --prefix=/usr

Build:

    $ make

Install:

    $ make install
    $ # or... to another root directory:
    $ make DESTDIR=/tmp/root/dir install

### Running ###

To route mavlink packets from master `ttyS1` to 2 other UDP endpoints, do as
following:

    $ mavlink-routerd -e 192.168.7.1:14550 -e 127.0.0.1:14550 /dev/ttyS1:1500000

The `1500000` after colon above on `/dev/ttyS1:1500000` is used to set the
UART baudrate. See more options with `mavlink-routerd --help`

It's also possible to route mavlinks packets from any interface using:

    $ mavlink-routerd -e 192.168.7.1:14550 -e 127.0.0.1:14550  0.0.0.0:24550

mavlink-router also listens, by default, port 5760 for TCP connections. Any
connection there will also receive routed packets.

<a name="Conffiles"></a>
### Conf file ###

It's also possible to use a .conf file to set options for mavlink-routerd.
By default, mavlink-routerd looks for a file
`/etc/mavlink-router/main.conf`. File location can be overriden via
`MAVLINK_ROUTER_CONF_FILE` environment variable, or via `-c` switch when running
mavlink-routerd.
An example of conf file can be found on [examples/config.sample](examples/config.sample)

#### Conf dirs ####

Besides default conf file, it's also possible to use a directory in where to
put some extra configuration files. Files on such directory will be read in
alphabetical order, and can add or override configurations found on previous
files.

By default, `/etc/mavlink-router/config.d` is the directory, but it can be
overriden via `MAVLINK_ROUTER_CONF_DIR` environment variable, or via `-d`
switch when running mavlink-routerd.

Suppose default configuration file defines an `UartEndpoint` using `Baud=56600`,
an example of overriding configuration would be:

```ini
[UartEndpoint bravo]
Baud = 115200
```

That would change `Endpoint bravo` baudrate to `115200`.

### Flight stack logging ###

Mavlink router can also collect flight stack log. It supports collecting
both PX4 and Ardupilot flight stacks logs. To start logging, set a
directory to `Log` key on `General` section of config file (or use
argument option `-l`).
Currently, mavlink router needs to be informed which MAVLink dialect
flight stack speaks, `common` or `ardupilotmega`. To define it, use
`MavlinkDialect` key. For instance, to collect Ardupilot logs to
`/var/log/flight-stack` directory, one could add to conf file:

    [General]
    Log=/var/log/flight-stack
    MavlinkDialect=ardupilotmega

Logs are collected on `.bin` (for Ardupilot) or `.ulg` (for PX4) files
inside specified directory. Note that they are named `XXXXX-date-time`,
where `XXXXX` is an increasing number.

For more information about configuration files, see [conf file section](#Conffiles).

### Contributing ###

Pull-requests are accepted on GitHub. Make sure to check coding style with the
provided script in tools/checkpatch and tools/checkpython, check for memory leaks
with valgrind and test on real hardware.

### Samples ###

Directory examples has some samples that can be used to test mavlink-router.
Those are Python scripts, and [pymavlink](https://github.com/ArduPilot/pymavlink)
is required.

#### Sender & receiver ####

One can test mavlink-router by using `examples/sender.py` and
`examples/receiver.py` to simulate traffic of mavlink messages.
First script send mavlink *ping* messages to a target mavlink system-id, and
second receives and respond them.
For instance:

    $ python examples/sender.py 127.0.0.1:3000 100 0

Will send mavlink *pings* to UDP port 300. Those pings will have `100` as
source system and will have `0` as target-system (`0` as target means broadcast).
Receiver could be set as:

    $ python examples/receiver.py 127.0.0.1:4000 50

Where `50` is receiver system id. Then, to route between those:

    $ mavlink-routerd -e 127.0.0.1:4000 0.0.0.0:3000

Note that it's possible to setup multiple senders and receivers to see
mavlink-router in action.
