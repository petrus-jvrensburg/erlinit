# erlinit

This is a replacement for `/sbin/init` that launches an Erlang/OTP release. It
is intentionally minimalist as it expects Erlang/OTP to be in charge of
application initialization and supervision. It can be thought of as a simple
Erlang/OTP release start script with some basic system initialization.

## Release setup

A system should contain only one Erlang/OTP release under the `/srv/erlang`
directory. A typical directory hierarchy would be:

    /srv/erlang
         ├── lib
         │   ├── my_app-0.0.1
         │   │   ├── ebin
         │   │   └── priv
         │   ├── kernel-2.16.2
         │   │   └── ebin
         │   └── stdlib-1.19.2
         │       └── ebin
         └── releases
             ├── 1
             │   ├── my_app.boot
             │   ├── my_app.rel
             │   ├── my_app.script
             │   ├── sys.config
             │   └── vm.args
             └── RELEASES

Currently, `erlinit` runs the Erlang VM found in `/usr/lib/erlang` so it is
important that the release and the VM match versions. As would be expected,
the `sys.config` and `vm.args` are used, so it is possible to configure
the system via files in the release. `erlinit` does not introduce any
new configuration files.

## Command line options

Linux passes all unknown kernel arguments to `init` or in this case, `erlinit`.
The following lists the options:

    -h Hang the system if Erlang exits. The default is to reboot.
    -s Run strace on Erlang
    -t Print out when erlinit starts and when it launches Erlang (for
       benchmarking)
    -u Remount the root filesystem with a unionfs
    -v Enable verbose prints

## Writable root file systems and unionfs

By default `erlinit` keeps the root filesystem mounted read-only. This is useful
since it significantly reduces the chance of corrupting the root filesystem at
runtime. If applications need to write to disk, they can always mount a
writable partition and have code that handles corruptions on it. Recovering from
a corrupt root filesystem is harder. During development, though, working with a
read-only root filesystem can be a pain so an alternative is to remount it
read-write. Applications can do this, but a better approach can be to remount it
using a unionfs. This way the root filesystem is kept read-only and all changes
are stored in memory until the next reboot. While changes are lost after a
reboot, you don't have to worry about a bad change bricking the target. If you
pass the '-u' option to `erlinit`, it will run the necessary commands to setup
the unionfs before starting Erlang. The Linux kernel must be patched with the
unionfs code for this to work. See http://unionfs.filesystems.org/ for
information.

## Debugging erlinit

Since `erlinit` is the first user process run, it can be a little tricky
to debug when things go wrong. Hopefully this won't happen to you, but if
it does, try passing '-v' in the kernel arguments so that `erlinit` runs in
verbose mode. If it looks like the Erlang runtime is being started, but it
crashes or hangs midway without providing any usable console output, try
passing '-s' in the kernel arguments and add the strace program to `/usr/bin`.
Sometimes you need to sift through the strace output to find the missing
library or file that couldn't be loaded. When debugged, please consider
contributing a fix back to help other `erlinit` users.
