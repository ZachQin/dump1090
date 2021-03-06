Dump1090-tty README
===

Dump 1090-tty is a Mode S decoder specifically designed for serial port devices.

Compared with dump1090, the main differences are:

* Replace RTL-SDR with serial port device.
* Support auto detect device name begin with "/dev/ttyS" and "/dev/ttyUSB".
* Support text file input (using —file command line).
* Network support trajectory stream (default port 30004). Trajectory message format: "!(flight),(longitude),(latitude),(altitude),(speed),(angle),(unix timestamp)\*" like "!CSN6909,115.9741,39.8630,10000,286,145,1510242849\*"

Installation
---

Type "make".

## Simple usage

Auto detect a serial port and use default baudrate 3,000,000 and no parity.

```
./dump1090tty
```

Normal usage
---

To capture traffic directly from your serial port device and show the captured traffic
on standard output with serial name "/dev/ttyUSB0", 3000000 baudrate and no parity, run the program:

    ./dump1090tty --name /dev/ttyUSB0 --speed 3000000

To just output hexadecimal messages:

    ./dump1090tty --name /dev/ttyUSB0 --speed 3000000 --raw

To run the program in interactive mode:

    ./dump1090tty --name /dev/ttyUSB0 --speed 3000000 --interactive

To run the program in interactive mode, with networking support, and connect
with your browser to http://localhost:8080 to see live traffic:

    ./dump1090tty --name /dev/ttyUSB0 --speed 3000000 --interactive --net

In interactive mode it is possible to have a less information dense but more
"arcade style" output, where the screen is refreshed every second displaying
all the recently seen aircrafts with some additional information such as
altitude and flight number, extracted from the received Mode S packets.

Using files as source of data
---

To decode data from file, use:

    ./dump1090tty --file /path/to/hexfile
It is possible to feed the program with data via standard input using
the --file option with "-" as argument.

Additional options
---

Dump1090-tty can be called with other command line options to set a different
gain, frequency, and so forth. For a list of options use:

    ./dump1090tty --help

Everything is not documented here should be obvious, and for most users calling
it without arguments at all is the best thing to do.

Reliability
---

By default Dump1090-tty tries to fix single bit errors using the checksum.
Basically the program will try to flip every bit of the message and check if
the checksum of the resulting message matches.

This is indeed able to fix errors and works reliably in my experience,
however if you are interested in very reliable data I suggest to use
the --no-fix command line switch in order to disable error fixing.

Performances and sensibility of detection
---

In my limited experience Dump1090-tty was able to decode a big number of messages
even in conditions where I encountered problems using other programs, however
no formal test was performed so I can't really claim that this program is
better or worse compared to other similar programs.

If you can capture traffic that Dump1090-tty is not able to decode properly, drop
me an email with a download link. I may try to improve the detection during
my free time (this is just an hobby project).

Network server features
---

By enabling the networking support with --net Dump1090-tty starts listening
for clients connections on port 30002 and 30001 (you can change both the
ports if you want, see --help output).

Port 30002
---

Connected clients are served with data ASAP as they arrive from the device
(or from file if --file is used) in the raw format similar to the following:

    *8D451E8B99019699C00B0A81F36E;

Every entry is separated by a simple newline (LF character, hex 0x0A).

Port 30001
---

Port 30001 is the raw input port, and can be used to feed Dump1090-tty with
data in the same format as specified above, with hex messages starting with
a `*` and ending with a `;` character.

So for instance if there is another remote Dump1090-tty instance collecting data
it is possible to sum the output to a local Dump1090-tty instance doing something
like this:

    nc remote-dump1090.example.net 30002 | nc localhost 30001

It is important to note that what is received via port 30001 is also
broadcasted to clients listening to port 30002.

It is possible to use Dump1090-tty just as an hub using --ifile with /dev/zero
as argument as in the following example:

    ./dump1090tty --net-only

Or alternatively to see what's happening on the screen:

    ./dump1090tty --net-only --interactive

Then you can feed it from different data sources from the internet.

Port 30003
---

Connected clients are served with messages in SBS1 (BaseStation) format,
similar to:

    MSG,4,,,738065,,,,,,,,420,179,,,0,,0,0,0,0
    MSG,3,,,738065,,,,,,,35000,,,34.81609,34.07810,,,0,0,0,0

This can be used to feed data to various sharing sites without the need to use another decoder.

Aggressive mode
---

With --aggressive it is possible to activate the *aggressive mode* that is a
modified version of the Mode S packet detection and decoding.
THe aggresive mode uses more CPU usually (especially if there are many planes
sending DF17 packets), but can detect a few more messages.

The algorithm in aggressive mode is modified in the following ways:

* Up to two demodulation errors are tolerated (adjacent entires in the magnitude
  vector with the same eight). Normally only messages without errors are
  checked.
* It tries to fix DF17 messages trying every two bits combination.

The use of aggressive mdoe is only advised in places where there is low traffic
in order to have a chance to capture some more messages.

Debug mode
---

The Debug mode is a visual help to improve the detection algorithm or to
understand why the program is not working for a given input.

In this mode messages are displayed in an ASCII-art style graphical
representation, where the individial magnitude bars sampled at 2Mhz are
displayed.

An index shows the sample number, where 0 is the sample where the first
Mode S peak was found. Some additional background noise is also added
before the first peak to provide some context.

To enable debug mode and check what combinations of packets you can
log, use `mode1090tty --help` to obtain a list of available debug flags.

Debug mode includes an optional javascript output that is used to visualize
packets using a web browser, you can use the file debug.html under the
'tools' directory to load the generated frames.js file.

How to test the program?
---

If you have an serial port device and you happen to be in an area where there
are aircrafts flying over your head, just run the program and check for signals.

However if you don't have an serial port device, or if in your area the presence
of aircrafts is very limited, you may want to try the sample file distributed
with the Dump1090-tty distribution under the "testfiles" directory.

Just run it like this:

    ./dump1090tty --file testfiles/hex.txt
Contributing
---

Dump1090tty was modified version of Dump1090. Delete the part of rtl-sdr and add the part of serial port.
