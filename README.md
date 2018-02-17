# qeman

A simple CLI manager for qemu setups

As the description says, this is a _simple_ CLI manager.
It does not do heavy error checking (it does basic error checking),
and cannot handle very advanced or exotic qemu setups. The supported
settings are listed/suggested below.

## Installation

The only dependency is bash.

~~Until I move `bash-ini-parser` into the `qeman` script file, that
is a dependency as well, and these two are required to be in the
same directory.~~

Just put the `qeman` script somewhere in your path. That's it.

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
[arch0]
localtime=enable
kvm=enable
boot_from_disk=disable
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

Note that in this example file `arch0` will not be able to run because
no hard drive is specified. Kernel and initrd qemu options are not
supported natively at the moment, but still may be speficied in the
`opts` setting.

## Writing the config file

This is honestly the most important section of qeman. Without knowing how to
make the config file, there is no use in the tool.

Using ini terminology, the section name corresponds to qeman's setup name.
Within each section, there are key-value pairs, where the keys are values are
separated by `=` and each pair is separated by newline.

The currently available keys are:
- `hda`
  - specifies the disk image file that will coresspond to `/dev/sda` in the VM
- `hdb`
  - specifies the disk image file that will coresspond to `/dev/sdb` in the VM
- `hdc`
  - specifies the disk image file that will coresspond to `/dev/sdc` in the VM
- `hdd`
  - specifies the disk image file that will coresspond to `/dev/sdd` in the VM
- `hde`
  - specifies the disk image file that will coresspond to `/dev/sde` in the VM
- `cd`
  - specifies the disk image file that will coresspond to `/dev/cdrom` or `/dev/sr0` in the VM
- `portfwd`
  - specifies port forwarding option
  - format: `[tcp|udp]::host_port-:vm_port`
  - automatically specifies netdev and device with `user,id=user.0` and `e1000`, so don't use this option if you want other network settings
- `mem`
  - specifies amount of memory for the VM
  - in megabytes - no need to specify units
- `localtime`
  - format: `enable|disable` - or simply nonexistant
  - is filled in automatically if `qeman setup new NAME` is used, because ini requires that each section have at least one member
- `kvm`
  - format: `enable|disable` - or simply nonexistant
  - specifies `-enable-kvm` - also automatically prepends `sudo`
- `boot_from_disk`
  - format: `enabme|disable` - or simply nonexistant
  - specifies `-boot d` - really this option should be called `boot_from_cd` instead
- `arch`
  - specifies the type of `qemu-system-` to execute
  - is required

There are a few special keys as well:
- `cmd`
  - this is a special option that allows you to specify an entire command to execute
  - if this is specified, all other options for the section will be ignored, and this cmd will be executed on `qeman run NAME`
- `opts`
  - this is a special option that allows you to specify arguments to pass to qemu
  - it is just a string (make sure they are double-quote delimited in the config) that will get appended to the qemu command on `qeman run NAME`
  - use this to specify your own network device, for example
- `prehook`
  - this is an entire command that will be executed before the qemu invokation itself
  - it is just a string (make sure they are double-quote delimited in the confg) that gets prepended before the qemu command and separated by semicolon on `qeman run NAME`
- `posthook`
  - this is an entire command that will be executed after the qemu invokation itself
  - it is just a string (make sure they are double-quote delimited in the confg) that gets appended after the qemu command and separated by semicolon on `qeman run NAME`

### Functions

(bold has been implemented)

#### run

- **qeman run [-bg] NAME**
  - runs qemu with settings as specified in the setups file
  - NAME selects the setup from the file
  - runs in foreground by default, unless `-bg` is specified

#### mode

- **qeman mode**
  - this will display current qeman mode
  - will return local or global
- **qeman mode local [save]**
  - this will set the current qeman mode to local
  - local mode means that qeman will autoload the `qeman.setups` file from the current working directory
  - the qeman mode setting is global, meaning that if you `cd` to another directory, the qeman mode will not change
  - if `save` is specified, then this mode setting will be saved to `./.qeman_mode`
    - this mode file has higher priority than the system-wide one - eg. if system-wide is global mode but local-mode is local, then setting will be local
    - delete this file if you want the system-wide qeman mode to be applied instead
- **qeman mode global [save]**
  - this will set the current qeman mode to global
  - global mode means that qeman will autoload the `qeman.setups` file from `$XDG_CONFIG_HOME/qeman/`, or if that fails, then from `$HOME/.config/qeman/`.
  - if `save` is specified, then this mode setting will be saved to `./.qeman_mode`
    - this mode file has higher priority than the system-wide one - eg. if system-wide is local mode but local-mode is global, then setting will be global
    - delete this file if you want the system-wide qeman mode to be applied instead

It should be noted that the `qeman.setups` file location can be overwritten by the environment variable `QEMAN_CONFIG_FILE`.

#### setup

Aliases have been added such that `setup` can be omitted. In this case `qeman setup ls` is equivalent to `qeman ls`, for example.

- **qeman setup new NAME**
  - adds a new section (ie. "setup") called NAME to the setups file
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
- **qeman setup rm NAME**
  - removes the section/setup called NAME including its children key-value pairs
- **qeman setup ls [-l]**
  - lists the current setups and their settings
  - only the setups in the config file specified by the mode will be considered. eg, if in local mode, the setups in the global config file will not be considered, and vice versa. If the config file is specified in `QEMAN_CONFIG_FILE`, then only those setups will be considered.
  - ~~currently, the only supported setup settings are the ones listed below in the `comp` section~~
    - all setttings will be displayed
  - by default only the list of setups will be displayed
    - if `-l` is specified then the settings of each setup will be displayed as well

#### comp

I'm not seeing any point in implementing these if users can just edit config files.
I'll document the config options better, though.

- qeman comp set/clear hdX NAME
- qeman comp set/clear cd NAME
- qeman comp set/clear portfwd [tcp|udp]::HOST_PORT-:GUEST_PORT
- qeman comp set/clear mem VALUE
- qeman comp enable/disable localtime
- qeman comp enable/disable kvm
- qeman comp enable/disable boot_from_disk
- qeman comp set/clear arch ARCH

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
