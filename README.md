
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

- qeman setup new NAME
- qeman setup load NAME
- qeman setup cp SRC DEST
- qeman setup mv SRC DEST
- qeman setup rm NAME
- **qeman setup ls**
  - lists the current setups and their settings
  - only the setups in the config file specified by the mode will be considered. eg, if in local mode, the setups in the global config file will not be considered, and vice versa.
  - currently, the only supported setup settings are the ones listed below in the `comp` section
- qeman setup pcs

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
