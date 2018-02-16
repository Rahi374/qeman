
### Functions

- qeman run NAME

- qeman mode
- qeman mode local
- qeman mode global

- qeman setup new NAME
- qeman setup load NAME
- qeman setup cp SRC DEST
- qeman setup mv SRC DEST
- qeman setup rm NAME
- qeman setup ls
- qeman setup pcs

- qeman comp add/rm hdX NAME
- qeman comp add/rm cd NAME
- qeman comp add/rm portfwd HOST_PORT:GUEST_PORT
- qeman comp mem VALUE
- qeman comp enable/disable localtime
- qeman comp enable/disable kvm
- qeman comp enable/disable boot_from_disk
- qeman comp arch ARCH

### Env vars

- TMPFS - defaults to /tmp, only neccesary if tmpfs not mounted at /tmp
- QEMAN_CONFIG_FILE - specify non-default config file location
