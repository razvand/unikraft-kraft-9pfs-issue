# kraft 9pfs Issue

This is a demonstration of [kraft 9pfs issue](https://github.com/unikraft/kraft/issues/71) when using 9PFS for filesystem support in Unikraft.
It runs on a QEMU/KVM Unikraft virtual machine.
A 9pfs configuration in the kraft.yaml file, similar to the `kraft.yaml` file in this repository, isn't used correctly by `kraft run`:

```
volumes:
  guest_fs:
    driver: 9pfs
```

## Expected Behavior

The `./guest_fs/` local directory is to be mounted as the root directory (`/`) inside the QEMU/KVM virtual machine.
It contains the `grass` file.
The program (`main.c`) reads the contents of the `/grass` file and prints it to standard output.

## Requirements to Reproduce the Issue

A QEMU/KVM environment for Unikraft is required.
Make sure you have QEMU installed and the GNU toolchain (GCC, Make) to build and run a Unikraft KVM image.

Install [kraft](https://github.com/unikraft/kraft), the latest version.
Make sure you have the kraft repositories downloaded by running:

```
$ kraft list update
```

## Steps

Configure the application:

```
$ kraft configure -m x86_64 -p kvm
```

Build the application:

```
$ kraft build
```

Everything runs OK when using the `qemu-guest` used as [a support script in kraft](https://github.com/unikraft/kraft/blob/staging/scripts/qemu-guest):

```
$ ./qemu-guest -e guest_fs/ -k ./build/unikraft-kraft-9pfs-issue_kvm-x86_64
[...]
o.   .o       _ _               __ _
Oo   Oo  ___ (_) | __ __  __ _ ' _) :_
oO   oO ' _ `| | |/ /  _)' _` | |_|  _)
oOo oOO| | | | |   (| | | (_) |  _) :_
 OoOoO ._, ._:_:_,\_._,  .__,_:_, \___)
                   Tethys 0.5.0~825b115
Hello, world!
File contents: The grass is green!
Bye, world!
```

Or when directly using the `qemu-system-x86_64` command as part of the `launch.sh` script:

```
$ ./launch.sh ./build/unikraft-kraft-9pfs-issue_kvm-x86_64
[...]
o.   .o       _ _               __ _
Oo   Oo  ___ (_) | __ __  __ _ ' _) :_
oO   oO ' _ `| | |/ /  _)' _` | |_|  _)
oOo oOO| | | | |   (| | | (_) |  _) :_
 OoOoO ._, ._:_:_,\_._,  .__,_:_, \___)
                   Tethys 0.5.0~825b115
Hello, world!
File contents: The grass is green!
Bye, world!
```

But it fails when using `kraft run`:

```
$ kraft run
[...]
[    0.100342] CRIT: [libvfscore] <rootfs.c @  122> Failed to mount /: 22
[    0.101749] ERR:  [libukboot] <boot.c @  102> Init function at 0x11dc10 returned error -1
```

## Fix

What worked for me was replacing [the troubling line](https://github.com/unikraft/kraft/blob/staging/kraft/app/app.py#L503) with:

```
            if volume.driver == VolumeDriver.VOL_9PFS.name:
```

With this update, `kraft run` works.
