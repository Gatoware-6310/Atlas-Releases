# Atlas-Releases
Atlas is a hobby operating system featuring:
- A Windows 95~ like GUI called Genesis (though it initially boots into TUI mode)
- A full filesystem in RAM
- USB drive support for persistent storage
- Several built in applications
- Support for userspace applications in Assembly, C, Lua, and GatoLang (with an onboard compiler for C, Assembly, and GatoLang
- And more!

Atlas is designed to work with Qemu and most real 64 bit hardware, though this remains largely untested. In Atlas, all applications have kernel-level permissions, allowing applications to essentially do whatever they want, which theoretically makes Atlas extremely customizable and extensible. Atlas comes with a graphics library designed to work with Genesis, and implements somewhat buggy userspace multitasking which is disabled by default. Multitasking with built-in applications, however, is largely flawless.

To run Atlas, install . The recommended launch command is:
```bash
qemu-system-x86_64   -enable-kvm   -cpu host   -machine q35   -m 8G   -drive file=atlas.iso,format=raw,if=ide   -device intel-hda   -device hda-duplex -device usb-kbd   -device usb-mouse
```
This includes a USB keyboard, USB mouse, 8 gigs of memory, and sound.

To launch Atlas in Qemu with a real USB drive (e.g, /dev/sdc on Linux), you can use this command:
```bash
qemu-system-x86_64   -enable-kvm   -cpu host   -machine q35   -m 8G   -drive file=atlas.iso,format=raw,if=ide   -device intel-hda   -device hda-duplex   -device qemu-xhci   -drive if=none,id=usb,file=/dev/sdc,format=raw,cache=none   -device usb-storage,drive=usb   -device usb-kbd   -device usb-mouse
```
You can use either the `
