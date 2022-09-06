# signmodules
A shell script for signing kernel modules with a Machine Owner Key, for Fedora.

# Background
In machines with Secure Boot installed, the bootloader, kernel, and all modules loaded by the kernel must be signed by either of the following:
- A hardware manufacturer
- A trusted Operating System vendor or provider

On Linux, a few distributions work with Secure Boot installed, as for example Fedora, Debian and Ubuntu. This means that they are trusted Operating System vendors, and hence the modules they ship as part of the distribution are signed by them and will work. 

However, if you install any software from a third party, that requires to load a kernel module, you will have to sign it yourself.

This script works for Fedora and will:

1. Sign all the modules passed as parameter, and all their dependencies, i.e., any other modules present in the same directory of the tree.
2. Restart any systemd unit that is failed, bringing back the system to a `running` state if all failed units are so due to kernel modules that have not been loaded.

The script should be run after a reboot performed as a result of a kernel update, or own / third party modules updates.

# Prerrequisites
- A computer with Secure Boot enabled, running Fedora.
- Create a Machine Owner Key and enroll it, following the relevant instructions on the Fedora [System Administrator's Guide: Signing Kernel Modules for Secure Boot](https://docs.fedoraproject.org/en-US/fedora/latest/system-administrators-guide/kernel-module-driver-configuration/Working_with_Kernel_Modules/#sect-signing-kernel-modules-for-secure-boot).
- The compression adn decompression utilities `xz` and `unxz` are also needed. They are provided by the package `xz`.

# Usage
Clone this repository or download the script [signmodules](https://github.com/gvisoc/signmodules/blob/main/signmodules), and place it in the `$PATH` variable.

## Invocation and Behaviour
The usage of the script is the following:

```
signmodules /path/to/MOK.priv /path/to/MOK.der MODULE_1 MODULE_2 ... MODULE_N
```
- `/path/to/MOK.priv`: private part of the MOK (Machine Owner Key --the signing key)
- `/path/to/MOK.der`: public part of the MOK.
- `MODULE_1 MODULE_2 ... MODULE_N`: a list of modules' names that we want to sign, e.g.: `vboxdrv v4l2loopback`

The script will ask for the user password, as both signing modules and restarting systemd services are operations that must be run under `sudo`.

Invoked like the above, the script won't write any logs (unless in trouble). Add `-v` (*verbose*) before the list of modules to enable logs

## Result Codes
The result codes are `0` if all operations were successful, or: 
- `1` if there were failures while signing modules, 
- `2` if there were failires while trying to restart systemd services,
- `3` if both.

## Example
An example invocation is the following, useful for signing all modules for *Virtual Box* and *Video4Linux Loopback*, and restarting the relevant systemd services, all in one go:
```
$ signmodules ~/Downloads/MOK.priv ~/Downloads/MOK.der -v vboxdrv v4l2loopback
[sudo] password for gvisoc: 
Passphrase for /home/gvisoc/Downloads/MOK.priv: 
[signmodules] Processing all modules for 'vboxdrv':
[signmodules]     Unpacking '/lib/modules/5.19.6-200.fc36.x86_64/extra/VirtualBox/vboxdrv.ko.xz'...
[signmodules]     Signing '/lib/modules/5.19.6-200.fc36.x86_64/extra/VirtualBox/vboxdrv.ko'...
[signmodules]     Recompressing '/lib/modules/5.19.6-200.fc36.x86_64/extra/VirtualBox/vboxdrv.ko'
[signmodules]     Unpacking '/lib/modules/5.19.6-200.fc36.x86_64/extra/VirtualBox/vboxnetadp.ko.xz'...
[signmodules]     Signing '/lib/modules/5.19.6-200.fc36.x86_64/extra/VirtualBox/vboxnetadp.ko'...
[signmodules]     Recompressing '/lib/modules/5.19.6-200.fc36.x86_64/extra/VirtualBox/vboxnetadp.ko'
[signmodules]     Unpacking '/lib/modules/5.19.6-200.fc36.x86_64/extra/VirtualBox/vboxnetflt.ko.xz'...
[signmodules]     Signing '/lib/modules/5.19.6-200.fc36.x86_64/extra/VirtualBox/vboxnetflt.ko'...
[signmodules]     Recompressing '/lib/modules/5.19.6-200.fc36.x86_64/extra/VirtualBox/vboxnetflt.ko'
[signmodules] Processing all modules for 'v4l2loopback':
[signmodules]     Unpacking '/lib/modules/5.19.6-200.fc36.x86_64/extra/v4l2loopback/v4l2loopback.ko.xz'...
[signmodules]     Signing '/lib/modules/5.19.6-200.fc36.x86_64/extra/v4l2loopback/v4l2loopback.ko'...
[signmodules]     Recompressing '/lib/modules/5.19.6-200.fc36.x86_64/extra/v4l2loopback/v4l2loopback.ko'
[signmodules] Restarting any failed services after processing modules:
[signmodules]     Requested 'vboxdrv' restart with result 0
[signmodules]     Requested 'systemd-modules-load' restart with result 0
[signmodules] Done. Exiting with code 0 --success.
$ echo $?
0
```
Without logs:
```
$ signmodules ~/Downloads/MOK.priv ~/Downloads/MOK.der -v vboxdrv v4l2loopback
[sudo] password for gvisoc: 
Passphrase for /home/gvisoc/Downloads/MOK.priv: 
$ echo $?
0
```

# Automation of the script
To run the script in an automation and without user interaction, set up the environment variable `KBUILD_SIGN_PIN` with the passphrase of the private part of the Machine Owner Key, `MOK.priv`, and run the script as an privileged service / under a system user with elevated privileges.

# Credits
This script is inspired by [this Gist](https://gist.github.com/reillysiemens/ac6bea1e6c7684d62f544bd79b2182a4) by [Reilly Tucker Siemens](https://gist.github.com/reillysiemens), and taken further to sign any set of modules, plus fix any broken systemd service caused by unsigned modules.
