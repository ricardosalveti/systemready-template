# Template Structure for SystemReady IR Compliance Reports
This repo structure is the template for collecting compliance evidence for a
SystemReady IR certification.

(For the ES and SR bands, see the [systemready-es-sr-template].)

[systemready-es-sr-template]: https://gitlab.arm.com/systemready/systemready-es-sr-template

## Branches
For IR 1.1 certification, git branch `ir1` of this template should be used.

## Verification script
A `check-sr-results.py` verification script is available in the [SystemReady
scripts] repository, which can verify if a directory structure is conforming to
this template.

[SystemReady scripts]: https://gitlab.arm.com/systemready/systemready-scripts

## General Instructions
General instructions for collecting SystemReady IR compliance logs:

### `./report.txt`
Fill in with information about the system being certified

### `./acs-console.log`
If using serial console, capture a full console log from first power on
until completion of ACS tests. Must include all output from firmware.
Should end at a Linux busybox shell prompt after running FWTS tests.

### `./acs_results/`
Place an entire copy of the `acs_results` partitions, as
follows:

    /app_output
    /fwts
    /linux_acs
    /linux_dump
    /sct_results
    /uefi
    /uefi_dump

Also include the `result.md` output from running the SCT results parser script
from https://gitlab.arm.com/systemready/edk2-test-parser.

### `./manual-results/`

Place any results from manually performed tests in this directory tree.
Manual test results must include both a console log of the commands
executed and the result files (if any) generated by the test.

### `./docs/`
Place any firmware or device documentation, manuals, user guides, build instructions, etc..

### `./fw/`
Place any firmware binaries directly there. See below for UEFI capsules details.

#### `./fw/screenshots/`
Place some screenshots showing the FW menus (such as UEFI Setup, BMC
web console, uboot shell) and any picture of the system under test with UART
connection.

#### `./fw/u-boot-sniff.log`
Run the following commands at U-Boot prompt and attach the logs:

    u-boot=> help
    u-boot=> version
    u-boot=> printenv
    u-boot=> printenv -e
    u-boot=> bdinfo
    u-boot=> rtc list
    u-boot=> sf probe
    u-boot=> usb reset
    u-boot=> usb info
    u-boot=> mmc rescan
    u-boot=> mmc list
    u-boot=> mmc info
    u-boot=> efidebug devices
    u-boot=> efidebug drivers
    u-boot=> efidebug dh
    u-boot=> efidebug memmap
    u-boot=> efidebug tables
    u-boot=> efidebug query
    u-boot=> efidebug boot dump
    u-boot=> efidebug capsule esrt
    u-boot=> bootefi hello ${fdtcontroladdr}
    u-boot=> bootefi selftest ${fdtcontroladdr}

#### `./fw/uefi-sniff.log`
Run the following commands in UEFI Shell and attach the logs (if not
already done by ACS). The UEFI Shell can be run from the ACS image by
pressing a key at the UEFI timeout prompt before tests begin to
execute.

    Shell> pci
    Shell> drivers
    Shell> devices
    Shell> devtree
    Shell> dmpstore
    Shell> dh -d -v
    Shell> memmap
    Shell> smbiosview

#### `./fw/capsule-update.log`
Demonstrate that `UpdateCapsule()` works by capturing a log of using
`CapsuleApp.efi` from the UEFI shell performing several attempts at installing a
different version of firmware.

Using the `capsule-tool.py` from the [SystemReady scripts], generate an unsigned
capsule and a tampered capsule from a signed capsule as follows:

```
$ capsule-tool.py --de-authenticate --output unauth.bin capsule1.bin
$ capsule-tool.py --tamper --output tampered.bin capsule1.bin
```

Start with a copy of the ACS image on an SD card or USB drive and put
a copy of the three capsules into the `BOOT` volume.
Setup the system in its original state.
Begin capture before releasing board from reset so that the initial
firmware log messages are captured.
Boot the ACS image, choose the default Grub option, and then press a
key to break out of running the ACS tests.
From the UEFI shell, use `CapsuleApp.efi` to attempt installing a new version
of the firmware with an unauthenticated capsule and with a tampered capsule.
Both attempts should fail and the system should not reboot:

```
Shell> fs2:\
FS2:\> EFI/BOOT/app/CapsuleApp.efi -D unauth.bin
FS2:\> EFI/BOOT/app/CapsuleApp.efi unauth.bin
FS2:\> EFI/BOOT/app/CapsuleApp.efi -D tampered.bin
FS2:\> EFI/BOOT/app/CapsuleApp.efi tampered.bin
```

Then use `CapsuleApp.efi` to install the new version of firmware with a signed
capsule.
`CapsuleApp.efi` should cause the board to reboot after installing.
The U-Boot console log should now show a different version of firmware.

```
Shell> fs2:\
FS2:\> EFI/BOOT/app/CapsuleApp.efi -D capsule1.bin
FS2:\> EFI/BOOT/app/CapsuleApp.efi capsule1.bin
```

#### `./fw/capsule-on-disk.log`
Demonstrate that delivery of capsules via file on mass storage device ("on
disk") works. When using this method, the firmware capsule is delivered as a
file, copied to the EFI System Partition (ESP) under the `EFI/UpdateCapsule`
folder.

Start with a copy of the ACS image on an SD card or USB drive and put
a copy of a signed capsule into the `BOOT` volume.
Setup the system in its original state.
Begin capture before releasing board from reset so that the initial
firmware log messages are captured.
Boot the ACS image, choose the default Grub option, and then press a
key to break out of running the ACS tests.
From the UEFI shell, use `CapsuleApp.efi` with the `-OD` option to install the
new version of firmware with a signed capsule using the "on disk" method.
`CapsuleApp.efi` should copy the capsule binary to the ESP and cause the board
to reboot.
Firmware update should be applied during system reboot and additional reboot
might happen.
The U-Boot console log should ultimately show a different version of firmware.

```
Shell> fs2:\
FS2:\> EFI/BOOT/app/CapsuleApp.efi -D capsule1.bin
FS2:\> EFI/BOOT/app/CapsuleApp.efi capsule1.bin -OD
```

#### `./fw/capsule*.bin`
Place UEFI signed capsule binaries under `./fw/`, with a name matching
`capsule*.bin`.

### `./os-logs/`

#### `./os-logs/linux-[distroname]-[distroversion]/`
Create a directory for each Linux distro used for testing.

Install the OS to a disk, and boot it.
Collect the installation and OS boot logs and save in single `console.log` text file.
The install log must begin when the platform is released from reset and
must include:

- All firmware output
- Output from Linux installer
- Output of reboot into installed OS.
- Output of following commands from Linux shell:

```
# dmesg
# lspci -vvv
# lscpu
# lsblk
# dmidecode
# uname -a
# cat /etc/os-release
# efibootmgr
# cp -r /sys/firmware ~/
# tar czf ~/sys-firmware.tar.gz ~/firmware
```

Make sure to use the intermediate copy step to capture the `/sys/firmware`
folder contents properly, then copy the resulting `sys-firmware.tar.gz` into the
results directory.

## SystemReady IR results example

After collecting the results the directory tree should look like this:

```
.
├── acs-console.log
├── acs_results/
│   ├── result.md
│   ├── app_output/
│   │   ├── CapsuleApp_ESRT_table_info.log
│   │   └── CapsuleApp_FMP_protocol_info.log
│   ├── fwts/
│   │   └── FWTSResults.log
│   ├── linux_acs/
│   │   └── bsa_acs_app
│   │       └── BSALinuxResults.log
│   ├── linux_dump/
│   │   └── lspci.log
│   ├── sct_results/
│   │   ├── EfiCompliantBBTest/
│   │   │   └── EfiCompliant.ini
│   │   ├── Overall/
│   │   │   ├── Summary.ekl
│   │   │   └── Summary.log
│   │   └── Sequence/
│   │       ├── EBBR_manual.seq
│   │       └── EBBR.seq
│   ├── uefi/
│   │   ├── BsaDevTree.dtb
│   │   └── BsaResults.log
│   └── uefi_dump/
│       ├── bcfg.log
│       ├── devices.log
│       ├── dh.log
│       ├── dmpstore.log
│       ├── drivers.log
│       ├── memmap.log
│       ├── pci.log
│       └── smbiosview.log
├── docs/
├── fw/
│   ├── readme.txt
│   ├── u-boot-sniff.log
│   ├── uefi-sniff.log
│   ├── capsule1.bin
│   ├── capsule-update.log
│   └── capsule-on-disk.log
├── manual-results/
├── os-logs/
│   ├── linux-distro1-version/
│   │   ├── console.log
│   │   └── sys-firmware.tar.gz
│   ├── linux-distro2-version/
│   │   ├── console.log
│   │   └── sys-firmware.tar.gz
│   └── screenshots/
├── README.md
└── report.txt
```
