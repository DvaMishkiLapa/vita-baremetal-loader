# PSVita bare-metal loader

## What's this?

This is a kernel plugin that lets you run bare-metal (i.e. without an OS beneath) payloads in ARMv7 non-secure System mode.

## How does it work?

At first, the plugin allocates a physically contiguous buffer where it loads the payload (`PAYLOAD_PATH` in `config.h`).
Then it triggers a power standby request and when PSVita OS is about to send the [Syscon](https://wiki.henkaku.xyz/vita/Syscon)
command to actually perform the standby, it changes the request type into a soft-reset and the resume routine address to a custom one (`resume.S`).

Once the PSVita wakes from the soft-reset, the custom resume routine identity maps the scratchpad (address [0x1F000000](https://wiki.henkaku.xyz/vita/Physical_Memory)) using a 1MiB section.
Then the payload bootstrap code (`payload_bootstrap.S`) is copied to the scratchpad and a jump is done to that location afterwards (passing some parameters such the payload physical address).

Since the payload bootstrap code is now in an identity-mapped location, it can proceed to disable the MMU and copy the payload from the previously allocated physically contiguous buffer to its destination address
(`PAYLOAD_PADDR` in `config.h`), to finally jump to it.

## Build

1. Make sure you have [VitaSDK](https://vitasdk.org/) installed and configured (try [vdpm](https://github.com/vitasdk/vdpm));
2. Change the path for the `payload.bin` download if you need it (`config.h:1`);
3. Depending on the firmware version on your PSVita, change the libraries you use (`Makefile:5-9`) ([source](https://gist.github.com/xerpi/5c60ce951caf263fcafffb48562fe50f?permalink_comment_id=4464046#gistcomment-4464046)):

   If you have firmware version **< 3.63** (*not verified, because I do not have such firmware*):

   ```makefile
   LIBS =	-ltaihenForKernel_stub -lSceSysclibForDriver_stub -lSceSysmemForDriver_stub \
	-lSceSysmemForKernel_stub -lSceThreadmgrForDriver_stub -lSceCpuForKernel_stub \
	-lSceCpuForDriver_stub -lSceUartForKernel_stub -lScePervasiveForDriver_stub \
	-lSceSysconForDriver_stub -lScePowerForDriver_stub -lSceIofilemgrForDriver_stub \
	-lSceSysrootForKernel_stub
   ```

    If you have firmware version **>= 3.63**:

   ```makefile
   LIBS =	-ltaihenForKernel_stub -lSceSysclibForDriver_stub -lSceSysmemForDriver_stub \
	-lSceSysmemForKernel_363_stub -lSceThreadmgrForDriver_stub -lSceCpuForKernel_363_stub \
	-lSceCpuForDriver_stub -lSceUartForKernel_363_stub -lScePervasiveForDriver_stub \
	-lSceSysconForDriver_stub -lScePowerForDriver_stub -lSceIofilemgrForDriver_stub \
	-lSceSysrootForKernel_stub
   ```

4. Build the project with the following commands: `make`.

## Installation

1. Copy your payload to your PSVita (default path is `ux0:linux/payload.bin`);
2. Copy `baremetal-loader.skprx` to your PSVita;
3. Load the plugin.

## Example bare-metal payload

Check [vita-baremetal-sample](https://github.com/xerpi/vita-baremetal-sample) as sample payload to be loaded with this plugin.

## Credits

Thanks to everybody, who contributed to the launch of the Linux kernel on PS Vita:

- [xerpi](https://github.com/xerpi);
- [Team Molecule](https://twitter.com/teammolecule) (formed by [Davee](https://twitter.com/DaveeFTW), Proxima, [xyz](https://twitter.com/pomfpomfpomf3), and [YifanLu](https://twitter.com/yifanlu));
- [TheFloW](https://twitter.com/theflow0)
- [motoharu](https://github.com/motoharu-gosuto)
- everybody at the [HENkaku](https://discord.gg/m7MwpKA) Discord channel;
- everybody who contributes to [wiki.henkaku.xyz](https://wiki.henkaku.xyz/) and helps reverse engineering the PSVita OS;
- [CreepNT](https://github.com/CreepNT) for improvements to this app.
