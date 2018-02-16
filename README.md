# qeman

A simple CLI manager for qemu setups

As the description says, this is a _simple_ CLI manager.
It does not do heavy error checking (it does basic error checking),
and cannot handle very advanced or exotic qemu setups. The supported
settings are listed/suggested below.

## Installation

The only dependency is bash.

Until I move `bash-ini-parser` into the `qeman` script file, that
is a dependency as well, and these two are required to be in the
same directory.

(`bash-ini-parser` came from <https://github.com/albfan/bash-ini-parser>)

## How it works

There is a single config file, known as the "setups file".
In practice, it is a single file called `qeman.setups`. It is in ini
format, where each section corresponds to a "setup", and the key-value
pairs in each section make up the settings for each setup. 

This file may be located anywhere, but by default in "local" mode
it is looked for in the current directory, and by default in "global"
mode it is looked for in `$XDG_CONFIG_HOME/qeman` and
`$HOME/.config/qeman`, in that order. The file path can always be
be specified in the environment variable `QEMAN_CONFIG_FILE`,
regardless of the mode.

Note that the mode is specified for the entire filesystem,
in `/tmp/qeman_mode`. If tmpfs is not mounted at `/tmp`, then the
tmpfs mountpoint must be specified in the environment variable
`TMPFS` without the trailing slash.

All qeman operations/comands are done on the setups file.

Example setups file:
```
[arch2]
hda=~/disk-images/hd/arch2.qcow2
hdb=~/disk-images/hd/hdrive.qcow2
cd=~/disk-images/cd/archlinux-2017.06.01-x86_64.iso
portfwd=tcp::2222-:22
mem=2048
localtime=enable
kvm=enable
boot_from_disk=enable
arch=x86_64
```

### Functions

(bold has been implemented)

#### run

- qeman run NAME

#### mode

- **qeman mode**
  - this will display current qeman mode
  - will return local or global
- **qeman mode local**
  - this will set the current qeman mode to local
  - local mode means that qeman will autoload the `qeman.setups` file from the current working directory
  - the qeman mode setting is global, meaning that if you `cd` to another directory, the qeman mode will not change
- **qeman mode global**
  - this will set the current qeman mode to global
  - global mode means that qeman will autoload the `qeman.setups` file from `$XDG_CONFIG_HOME/qeman/`, or if that fails, then from `$HOME/.config/qeman/`.

It should be noted that the `qeman.setups` file location can be overwritten by the environment variable `QEMAN_CONFIG_FILE`.

#### setup

- **qeman setup new NAME**
  - adds a new section called NAME (ie. "setup") to the setups file
  - enables `localtime` by default, since the ini parser will break if there's no key-value pairs in a section :/
    - `localtime` was chosen as the default because I anticipated that this is probably the best (most useful and least detrimental) "default" option
  - `kvm` and `boot_from_disk` will appear as disabled; this is because these two (along with `localtime`) are assumed to be disabled if they are not specified.
- **qeman setup cp SRC DEST**
  - copies the section SRC to a new section called DEST, and copies over all the settings as well
  - all these changes are done within the same setups file
- **qeman setup mv SRC DEST**
  - renames the section SRC to section DEST
  - that's it. Also, there's no overwriting; only simple renaming.
  - moving setups between setups files is another thing that this command does *not* do
- qeman setup rm NAME
- **qeman setup ls**
  - lists the current setups and their settings
  - only the setups in the config file specified by the mode will be considered. eg, if in local mode, the setups in the global config file will not be considered, and vice versa. If the config file is specified in `QEMAN_CONFIG_FILE`, then only those setups will be considered.
  - currently, the only supported setup settings are the ones listed below in the `comp` section

#### comp

- qeman comp add/rm hdX NAME
- qeman comp add/rm cd NAME
- qeman comp add/rm portfwd [tcp|udp]::HOST_PORT-:GUEST_PORT
- qeman comp mem VALUE
- qeman comp enable/disable localtime
- qeman comp enable/disable kvm
- qeman comp enable/disable boot_from_disk
- qeman comp arch ARCH

### Env vars

- TMPFS - defaults to /tmp, only neccesary if tmpfs not mounted at /tmp
- QEMAN_CONFIG_FILE - specify non-default config file location


## Undefined Behavior

ie. "I haven't tested it, I'm probably not going to,
and it's likely to do something
unanticipated (aka bad), so don't blame me if you do it and
something breaks!"

- Have duplicate section/setup names
- section/setup lines' opening square bracket not being flush left

## Rationale

I like qemu, but I found that the commands were continuously
getting longer and harder, if even possible, to remember.
I found myself always doing reverse-i-search for `qemu` until I
got the right one. I eventually wrote a script, but really, it
was just one of my long commands with the drive name swapped out
for a command line argument. Then I figured it might be nice to have
a vm manager. I'm fairly certain one exists out there somewhere, but
I guess I didn't look hard enough, and I wanted something minimalist,
so I decided to write my own, and this is it.
