# Template Structure for SystemReady Compliance Reports
This repo structure is the template for collecting compliance evidence for a
SystemReady certification.

## Verification script
A `check-sr-results.py` verification script is available in the [SystemReady
scripts] repository, which can verify if a directory structure is conforming to
this template.

[SystemReady scripts]: https://gitlab.arm.com/systemready/systemready-scripts

## General Instructions
General instructions for collecting SystemReady compliance logs:

### `./report.txt`
Fill in with information about the system being certified

### `./acs-console.log`
If using serial console, capture a full console log from first power on
until completion of ACS tests. Must include all output from firmware.
Should end at a Linux busybox shell prompt after running FWTS tests.

### `./acs_results/`
Place an entire copy of the `LUV-Results` or `acs_results` partitions, as
follows:

For Enterprise ACS, under the `LUV-Results` partition:

    /luv-results-<date>
        /sbsa_results
            /uefi
            /linux
        /SCT_RUN
        /sdei_results

For ACS-ES and ACS-IR, under the `acs_results` partition:

    /fwts
    /linux
    /linux_dump
    /sct_results
    /uefi
    /uefi_dump

For ACS-IR, also include the `result.md` output from running the SCT results
parser script from https://gitlab.arm.com/systemready/edk2-test-parser.

### `./manual-results/`

Place any results from manually performed tests in this directory tree.
Manual test results must include both a console log of the commands
executed and the result files (if any) generated by the test.

### `./manual-results/bsa-linux/`

Place any BSA or SBSA results collected manually from Linux.

#### Enterprise-ACS Examples:

SBSA Linux results can be collected by running the following commands
after booting LUVOS:

    insmod /lib/modules/4.18.0-luv/extra/sbsa_acs.ko
    sbsa -l 3

You can include verbose output (-v):

    sbsa -l 3 -v 2

To skip a specific problematic test, use -skip <testnum>. For example:

    sbsa -l 3 -v 2 -skip 20,36

### `./manual-results/bsa-uefi/`

Place any BSA or SBSA results collected manually from UEFI Shell

#### Enterprise-ACS Examples:

SBSA UEFI results can be collected by running the following
commands after booting LUVOS:

    sbsa.efi -l 3

You can include verbose output (-v):

    sbsa.efi -l 3 -v 2

To skip a specific problematic test, use -skip <testnum>

    sbsa.efi -l 3 -v 2 -skip 407

### `./manual-results/fwts/`
Place any additional FWTS results collected manually by running:

    fwts -r stdout -q –sbbr

### `./manual-results/sct/`
Place any additional SCT results collected manually

    FS2:\SCT>SCT.efi -a –v

If there are issues with specific cases, can skip them by running the UI with sct -u
and selecting/unselecting specific test cases


### `./docs/`
Place any firmware or device documentation, manuals, user guides, build instructions, etc..

### `./fw/`

#### `./fw/screenshots/`
Place some screenshots showing the FW menus (such as UEFI Setup, BMC
web console, uboot shell)

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
    u-boot=> efidebug boot dump
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
    Shell> acpiview -l
    Shell> acpiview
    Shell> acpiview -r 2
    Shell> acpiview -s DSDT -d  (if DSDT table is present)
    Shell> acpiview -s SSDT -d (if SSDT table is present)
    Shell> CapsuleApp.efi -P
    Shell> CapsuleApp.efi -E

#### `./fw/capsule-update.log`
Demonstrate that `UpdateCapsule()` works by capturing a log of using
CapsuleApp.efi from the UEFI shell to install a different version of
firmware.

Start with a copy of the ACS image on an SD card or USB drive and put
a copy of the firmware capsule into the `BOOT` volume.
Begin capture before releasing board from reset so that the initial
firmware log messages are captured.
Boot the ACS image, choose the default Grub option, and then press a
key to break out of running the ACS tests.
From the UEFI shell, use CapsuleApp.efi to install the new version
of firmware.
CapsuleApp.efi will cause the board to reboot after installing.
The U-Boot console log should now show a different version of firmware.

```
Shell> fs2:\
FS2:\> efi/boot/app/capsuleapp.efi capsule.bin
```

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
dmesg
lspci -vvv
lscpu
lsblk
dmidecode
uname -a
efibootmgr
tar cfz sys-firmware.tar.gz /sys/firmware
```

Copy the resulting `sys-firmware.tar.gz` into the results directory.

#### `./os-logs/esxi/`
Install VMWare ESXi to a disk, and boot it.
Connect to vSphere web console over network and capture screenshots.
Run the following commands and attach the logs:

```
dmesg
lspci
irqinfo
localcli hardware pci list
localcli storage core adapter list
```

#### `./os-logs/winpe/`
Boot WinPE from USB key, or Install Windows a disk, and boot it.
Include screenshots and video of WinPE booting (from FW menu to OS startup,
and keyboard working in command prompt)

Run the following commands and attach the logs:

```
pnputil/enum-devices
pnputil/enum-drivers
Systeminfo
ver
```

## Band Specific Examples

## SystemReady IR results

For IR, after collecting the results the directory tree should look like this:

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
│   │   └── BsaResults.log
│   └── uefi_dump/
│       ├── acpiview_l.log
│       ├── acpiview.log
│       ├── acpiview_r.log
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
│   └── u-boot-sniff.log
├── manual-results/
├── os-logs/
│   ├── esxi/
│   ├── linux-distro1-version/
│   │   ├── console.log
│   │   └── sys-firmware.tar.gz
│   ├── linux-distro2-version/
│   │   ├── console.log
│   │   └── sys-firmware.tar.gz
│   ├── screenshots/
│   └── winpe/
├── README.md
└── report.txt

19 directories, 28 files
```
