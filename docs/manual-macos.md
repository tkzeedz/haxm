## Building for macOS
### Prerequisites
* Xcode 7.2.1 or later
* OS X 10.10 SDK (archived [here][osx-sdks])
  * Use `xcodebuild -showsdks` to list installed SDKs.
  * It is also possible to build HAXM using a newer version of macOS SDK.
However, using an older SDK ensures that the generated kext is compatible with
older versions of macOS.
* NASM 2.11 or later
  * Install to `/usr/local/bin/` using Homebrew: `brew install nasm`
  * Note that Apple NASM (`/usr/bin/nasm`) cannot be used.

### Build steps
1. `cd /path/to/haxm/`
1. `cd platforms/darwin/`
1. `xcodebuild -configuration Debug`
   * Use `-sdk` to override the default macOS SDK version (10.10), e.g.
`-sdk macosx10.12` (to select SDK 10.12) or even `-sdk macosx` (to select the
latest SDK installed).
   * Use `Release` instead of `Debug` to build an optimized kext that is
suitable for release.

If successful, the kext (`intelhaxm.kext/`) will be generated in
`/path/to/haxm/platforms/darwin/build/Debug/`.

## Testing on macOS
### System requirements
Note that these are requirements for the _test_ environment, which does not
have to be the same as the _build_ environment.

1. Hardware requirements are the same as those for Windows.
1. OS X 10.10 or later.

### Loading and unloading the test kext
The `intelhaxm.kext` generated by the `Debug` and `Release` build configurations
is not signed. Unless you can sign it using a special kind of Apple Developer ID
Certificate, you must configure your test Mac to allow unsigned kexts to load:
* For OS X 10.10, the solution is to add the `kext-dev-mode=1` _boot-arg_. More
details can be found [here][macos-kext-dev-mode].
* For macOS 10.11 and later, the solution is to turn off System Integrity
Protection (SIP). More details can be found [here][macos-sip-disable].

To load the test kext:
1. Make sure no other HAXM kext is loaded. If the output of
`kextstat | grep intelhaxm` is not empty, you must unload the existing HAXM kext
first: `sudo kextunload -b com.intel.kext.intelhaxm`
1. `sudo chown -R root:wheel /path/to/intelhaxm.kext`
1. `sudo chmod -R 755 /path/to/intelhaxm.kext`
1. `sudo kextload /path/to/intelhaxm.kext`
   * Note that `kextload` can load a kext from any folder, so there is no need
to copy the test kext to `/Library/Extensions/` first.

To unload the test kext:
1. `sudo kextunload /path/to/intelhaxm.kext`
1. Optionally, you may want to restore the original HAXM kext, which is usually
the signed one installed to `/Library/Extensions/`:
`sudo kextload /Library/Extensions/intelhaxm.kext`

### Viewing kext logs
On macOS, HAXM debug output goes to the system log database, and can be
retrieved at almost any time.

* On OS X 10.10, HAXM log messages are written immediately to
`/var/log/system.log`. You can monitor this file for real-time updates using
_Console.app_ or the `syslog -w` command.
* On macOS 10.11 or later, HAXM log messages are no longer written to
`/var/log/system.log`, and there is no good way to capture them in real time.
However, you can still retrieve them at a later time using one of the following
methods:
  1. `log show --predicate 'sender == "intelhaxm"' --style syslog --last 1h`,
which is complex but very flexible. In this example, `--last 1h` indicates the
past hour, and can be replaced with other queries.
  1. `sudo dmesg | grep hax`, which is simple, but does not show the timestamp
of each message.

[osx-sdks]: https://github.com/phracker/MacOSX-SDKs
[macos-kext-dev-mode]: https://developer.apple.com/library/archive/documentation/Security/Conceptual/System_Integrity_Protection_Guide/KernelExtensions/KernelExtensions.html
[macos-sip-disable]: https://developer.apple.com/library/archive/documentation/Security/Conceptual/System_Integrity_Protection_Guide/ConfiguringSystemIntegrityProtection/ConfiguringSystemIntegrityProtection.html