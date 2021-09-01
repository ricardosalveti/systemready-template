# Template Structure for SystemReady Compliance Reports

This repo structure is the template for collecting compliance evidence for a
SystemReady certification.

## General Instructions

General instructions for collecting SystemReady compliance logs:

./report.txt
    Fill in with information about the system being certified

./acs
    If using serial console, capture a full console log from first power on
    until completion of ACS tests. Must include all output from firmware.
    Should end at a Linux busybox shell prompt after running FWTS tests.

./acs/acs-results
    Place an entire copy of the 'LUV-Results' or 'acs-results' partitions, as
    follows:

        For Enterprise ACS, under the 'LUV-Results' partition:
            /luv-results-<date>
            /sbsa_results
                /uefi
                /linux
            /SCT_RUN
            /sdei_results

        For ACS-ES and ACS-IR, under the 'acs-results' partition:
            /fwts
            /linux
            /linux_dump
            /sct_results
            /uefi
            /uefi_dump

./acs/bsa-linux

    Place any BSA or SBSA results collected manually, from UEFI or Linux.

    Examples:

    Enterprise-ACS:

        SBSA Linux results can be collected by running the following commands
        after booting LUVOS:
            insmod /lib/modules/4.18.0-luv/extra/sbsa_acs.ko
            sbsa -l 3

        You can include verbose output (-v):
            sbsa -l 3 -v 2

        To skip a specific problematic test, use -skip <testnum>. For example:
            sbsa -l 3 -v 2 -skip 20,36

./acs/bsa-uefi

    Place any BSA or SBSA results collected manually from UEFI Shell

    Examples:

    Enterprise-ACS

        SBSA UEFI results can be collected by running the following commands
        after booting LUVOS:
            sbsa.efi -l 3

        You can include verbose output (-v):
            sbsa.efi -l 3 -v 2

        To skip a specific problematic test, use -skip <testnum>
            sbsa.efi -l 3 -v 2 -skip 407

./acs/fwts
    Place any additional FWTS results collected manually by running:
        fwts -r stdout -q –sbbr

./acs/sct
    Place any additional SCT results collected manually
        FS2:\SCT>SCT.efi -a –v

    If there are issues with specific cases, can skip them by running the UI with sct -u
    and selecting/unselecting specific test cases


./docs
    Place any firmware or device documentation, manuals, user guides, build instructions, etc..


./fw
    ./fw/screenshots

        Place some screenshots showing the FW menus (such as UEFI Setup, BMC
        web console, uboot shell)

    ./fw/u-boot

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



    ./fw/uefi_shell
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

./os_logs

    ./os_logs/linux_[distroname]_[distroversion]

        Create a directory for each Linux distro used for testing.

        Install the OS to a disk, and boot it.
        Collect the installation and OS boot logs and save in single text file.
        The install log must begin when the platform is released from reset and
        must include:
        - all firmware output
        - output from Linux installer
        - reboot into installed OS.
        - Output of following commands from Linux shell:

            dmesg
            lspci -vvv
            lscpu
            lsblk
            dmidecode
            uname -a
            efibootmgr
            copy the entire content of /sys/firmware and attach zipped/archive file

    ./os_logs/esxi

        Install VMWare ESXi to a disk, and boot it
            Connect to vSphere web console over network and capture screenshots
            Run the following commands and attach the logs:
                dmesg
                lspci
                irqinfo
                localcli hardware pci list
                localcli storage core adapter list


    ./os_logs/winpe

        Boot WinPE from USB key, or Install Windows a disk, and boot it
        Include screenshots and video of WinPE booting (from FW menu to OS startup,
        and keyboard working in command prompt)

        Run the following commands and attach the logs:
            pnputil/enum-devices
            pnputil/enum-drivers
            Systeminfo
            ver
