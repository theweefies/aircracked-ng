#Install instructions

```bash
sudo apt-get update

sudo apt-get install libtool m4 automake pkg-config

git clone https://github.com/theweefies/aircracked-ng

cd aircracked-ng

autoreconf -i

./configure

make

make install

sudo vim /etc/profile

#add the following line to the end
PATH=$PATH:/usr/local/lib

reboot

sudo ldconfig
```

# Aircracked-ng changes

This version of the aircrack suite contains changes to aireplay-ng, airodump-ng:

airodump-ng:
- NEW FEATURES 
    - ppi/radiotap creation:   Pass the -p/--ppi option with --gpsd (or not if using --coords) to create radiotap/ppi geo headers. Currently, the TSF Timer, frequency, RSSI, noise, and rate are being pulled from ri structure for radiotap data. Some of these may need to be modified or adjusted (specifically the rate) due to my misconceptions or misunderstanding of the existing code. The GPS information used for geotagging is the lat, lon, and altitude. May add speed as well.
    - 802.11ax/6E support:     Pass the -X/--80211ax option to use standard 6E primary channel numbers with the -c option. Also added 'x' character to accepted bands with use of the --band option. Supports use of other bands, a, b, g by using freq mappings for other bands and using frequency scanning mode.
    - Targeting mode:          Pass the -z/--target to use targeting mode. Option can accept single mac address or file of new-line separated MACs. Will highlight identified MAC in AP or STA display tables. 
    - Fixed Coordinates:       Pass the -y/--coords option with coordinates in the format 35.121212,-75.232323 (floating point, signed, comma separated). These will be encoded into into the ppi geotag.
    - src/airodump-ng/airodump-ng.c:
      line:
        - 60:          added <ctype.h> include (i think??)
        - 130:         ax_all_chans array
        - 143:         ax_chans array
        - 151:         channel_frequency_map_bg
        - 170:         channel_frequency_map_a
        - 198:         channel_frequency_map_ax
        - 263:         letoh24 - function to convert a 3 byte array to host order
        - 277:         frequency #defines
        - 280-433:     ppi header functions/vars/#defines
        - 582:         4x lopt additions - int scan_11ax, int ppi, double coordinates[2], and int target
        - 588-651:     targeting globals
        - 1114:        modified usage table
        - 1787:        Set some of the 802.11ax properties to zero for new ap_cur
        - 2401-2475:   Ext. Tag and HE Operation parsing for AX
        - 3507-3567:   PPI writing calls and logic for encoding optional fixed coords
        - 4226-4231:   Target AP identification and marking
        - 4546-4549:   Added text fg color default after marking AP target (if in target mode)
        - 4641-4651:   Target STA identification and marking
        - 4715-4717:   Added text fg color default after marking STA target (if in target mode)
        - 4730-4732:   Added text fg color default at end of processing loop to ensure that text resets to ad default (if in target mode)
        - 4740: TO DO: Maybe update Target NA Station identification and marking?
        - 5747-5846:   Added functions dealing with converting channels to frequency strings
        - 5848:        modified invalid_channel to account for ax channels
        - 5924,5957:   modified getchannels to account for ax channels
        - 6416:        updated freq array to hold 3x options (a, bg, ax)
        - 6424:        added freq_string to hold updated frequencies
        - 6492:        updated long_options array (4x) - 80211ax, ppi, coords, target
        - 6583:        set new lopts to 0 (5x), lopt.scan_11ax, lopt.target, lopt.ppi, lopt.coordinates[0], lopt.coordinates[1]
        - 6687:        updated short options string, fix to this was left a colon on end of X. Options   added: Xpzy:
        - 6785:        added case 'X' for 802.11ax
        - 6791:        modified case 'c' for ax parsing; should retain default behavior if -X is not passed
        - 6860:        modified case 'b' for ax band use 
        - 6912:        added case 'z' for target mac highlighting
        - 6924:        added case 'y' for fixed coordinates option
        - 7488:        added logic to ensure that gpsd option is passed with ppi option (Unless fixed coordinates option is in play; however, it may be desirable to just build the radiotap - this functionality already exists)
        - 7544:        added argument ppi to dump_initialize_multi_format call

    - include/aircrack-ng/support/station.h:
      line:
        - 85: added ax_channel_info structure. This definitely needs to be more robust, but it serves its purpose.
        - 122: added ax_channel declaration
        - 230: added marked and marked_color to ST_info

    - include/aircrack-ng/support/communications.h:
      line:
        - 374: added ppi argument to dump_initialize_multi_format function signature

    - lib/libac/support/communications.c:
      line:
        - 959: added ppi argument to dump_initialize_multi_format
        - 1139: added logic test for ppi, so that if we are writing a file with ppi headers we change the linktype in the global header
        - 1207: added a zero value 3rd argument (ppi) to the return call for dump_initialize, since airodump doesn't use the dump_initialize call, and other programs in the suite might (did a in-file grep and don't see any specific issues this might cause)

    - FIXES - 18 MAR 2024:
      - 588-651:    - changed MAC address/targeting-related function signatures to static
      - 624-641:    - Modified parseMACAddressFile to ensure that improperly formatted lines (specifically lines longer than a mac address) are treated in such a manner that the additional characters are read and ignored until a new line character is reached (or end of file). This ensures the next line is read at the start. Additionally, the array size of MAX_TARGETS is protected.
      - 1787:       - Set some of the 802.11ax properties to zero for new ap_cur
      - 2401-2475:  - Added more boundary checking through Ext tag (HE Operation) parsing. also added    letoh24 to ensure endianness for different architectures (This includes compiler ifdef block that will add code if big endian to swap bytes, otherwise no swap). This should still be tested on big-endian systems. The arithmetic changes for determining the 6GHz Operation Information presence should be sound. Additionally, a switch and case statement was employed and proper channel width determination was employed based on the 802.11ax-2021 standard. I did not turn this into a function as you originally mentioned in your email - this would be a departure from the way that every other tag is processed in this portion of the code - i can easily make the change, but i wanted confirmation that is what you actually want before doing it.
      - 5747-5846:  - Added static declaration for channel/frequency mapping functions. Fixed channel_to_frequency_ax: get first and last channel from channel_frequency map and perform boundary check against channel argument. The other functions that perform channel_to_freq_string operations have an inherent boundary check in the inner 'for' loop (loop until -1, which is the end element of each of these arrays). Fixed channels_to_freq_string_ax to respect the boundaries of freq_string and avoid writing beyond its allocated space. Additionally, modified channels_to_freq_string_a (and bg) to ensure that the boundary of freq_string is respected and protected against a buffer overflow.
      - 6687:       - Added colon after option 'z' in short options string to indicate takes argument(oops).

aireplay-ng:
- UPDATED 
    - Deauthentication:   Reason code default is now set to reason code 1. Deauthentication frames specified in the count are true to the count, except in instances where the interface is shared with airodump.
                                When a second card/interface is used for airodump or the interface is not shared, the number of frames transmitted is accurate to the count, both for broadcast and directed deauths.
- NEW FEATURE 
    - Probe Requests: A new option (-P / --probe) has been added to the aireplay-ng binary. This option takes a positive integer for the count, similar to the deauth option. You can pass an
                                essid or bssid to send directed probe requests, or nothing to send probe requests and elicit responses from any surrounding APs. Of note: this script first conducts a
                                hearability check by sending a broadcast probe at intervals until a reponse is received to confirm that AP(s) are present, and then sends the number of directed probes    
                                specified by the count. The same note above about interface sharing remains true for sending probes as well.
    - src/aireplay-ng/aireplay-ng:
      line:
        - 183: changed default rc to 1 in usage print statement
        - 197: added -P/--probe option to send a probe request and listen for a response; purpose is to solicit make/model information in response.
        - 447-531: removed for loop with hard-coded 64 count deauth. now sends only the count that the user specifies
        - 5060: added do_probe function to send probes
        - 6380: updated opt.deauth_rc default to 1. This is operationally the most effective rc.
        - 6411: updated long_options for --probe option
        - 6424: update short options string
        - 6869: added 'P' case statement
        - 7180: added case return statement for 'P'

# Aircrack-ng

[![Alpine Linux Build Status](https://buildbot.aircrack-ng.org/badges/aircrack-ng-alpine.svg?left_text=Alpine%20Linux%20Build)](https://buildbot.aircrack-ng.org/)
[![Kali Linux Build Status](https://buildbot.aircrack-ng.org/badges/aircrack-ng-kali.svg?left_text=Kali%20Linux%20Build)](https://buildbot.aircrack-ng.org/)
[![Armel Kali Linux Build Status](https://buildbot.aircrack-ng.org/badges/aircrack-ng-armel.svg?left_text=Armel%20Kali%20Linux%20Build)](https://buildbot.aircrack-ng.org/)
[![Armhf Kali Linux Build Status](https://buildbot.aircrack-ng.org/badges/aircrack-ng-armhf.svg?left_text=Armhf%20Kali%20Linux%20Build)](https://buildbot.aircrack-ng.org/)
[![DragonFly BSD Build Status](https://buildbot.aircrack-ng.org/badges/aircrack-ng-dfly.svg?left_text=DragonFly%20Build)](https://buildbot.aircrack-ng.org/)
[![FreeBSD 11 Build Status](https://buildbot.aircrack-ng.org/badges/aircrack-ng-fbsd-11.svg?left_text=FreeBSD%2011%20Build)](https://buildbot.aircrack-ng.org/)
[![FreeBSD 12 Build Status](https://buildbot.aircrack-ng.org/badges/aircrack-ng-fbsd-12.svg?left_text=FreeBSD%2012%20Build)](https://buildbot.aircrack-ng.org/)
[![OpenBSD 6 Build Status](https://buildbot.aircrack-ng.org/badges/aircrack-ng-obsd.svg?left_text=OpenBSD%20Build)](https://buildbot.aircrack-ng.org/)
[![NetBSD 8.1 Build Status](https://buildbot.aircrack-ng.org/badges/aircrack-ng-netbsd81.svg?left_text=NetBSD%20Build)](https://buildbot.aircrack-ng.org/)
[![Coverity Scan Build Status](https://scan.coverity.com/projects/aircrack-ng/badge.svg)](https://scan.coverity.com/projects/aircrack-ng)
[![PackageCloud DEB](https://img.shields.io/badge/deb-packagecloud.io-844fec.svg)](https://packagecloud.io/aircrack-ng/git/install#bash-deb)
[![PackageCloud RPM](https://img.shields.io/badge/rpm-packagecloud.io-844fec.svg)](https://packagecloud.io/aircrack-ng/git/install#bash-rpm)

Aircrack-ng is a complete suite of tools to assess WiFi network security.

It focuses on different areas of WiFi security:
 * Monitoring: Packet capture and export of data to text files for further processing by third party tools.
 * Attacking: Replay attacks, deauthentication, fake access points and others via packet injection.
 * Testing: Checking WiFi cards and driver capabilities (capture and injection).
 * Cracking: WEP and WPA PSK (WPA 1 and 2).

All tools are command line which allows for heavy scripting. A lot of GUIs have taken advantage of this feature. It works primarily on Linux but also Windows, macOS, FreeBSD, OpenBSD, NetBSD, as well as Solaris and even eComStation 2. 

# Building

## Requirements

 * Autoconf
 * Automake
 * Libtool
 * shtool
 * OpenSSL development package or libgcrypt development package.
 * Airmon-ng (Linux) requires ethtool, usbutils, and often pciutils.
 * On Windows, cygwin has to be used and it also requires w32api package.
 * On Windows, if using clang, libiconv and libiconv-devel
 * Linux: LibNetlink 1 or 3. It can be disabled by passing --disable-libnl to configure.
 * pkg-config (pkgconf on FreeBSD)
 * FreeBSD, OpenBSD, NetBSD, Solaris and OS X with Macports: gmake
 * Linux/Cygwin: make and Standard C++ Library development package (Debian: libstdc++-dev)

Note: Airmon-ng only requires pciutils if the system has a PCI/PCIe bus and it is populated.
      Such bus can be present even if not physically visible. For example, it is present,
      and populated on the Raspberry Pi 4, therefore pciutils is required on that device.

## Optional stuff

 * If you want SSID filtering with regular expression in airodump-ng
   (-essid-regex) PCRE development package is required.
 * If you want to use airolib-ng and '-r' option in aircrack-ng,
   SQLite development package >= 3.3.17 (3.6.X version or better is recommended)
 * If you want to use Airpcap, the 'developer' directory from the CD/ISO/SDK is required.
 * In order to build `besside-ng`, `besside-ng-crawler`, `easside-ng`, `tkiptun-ng` and `wesside-ng`,
   libpcap development package is required (on Cygwin, use the Airpcap SDK instead; see above)
 * rfkill
 * If you want Airodump-ng to log GPS coordinates, gpsd is needed
 * For best performance on SMP machines, ensure the hwloc library and headers are installed. It is strongly recommended on high core count systems, it may give a serious speed boost
 * CMocka for unit testing
 * For integration testing on Linux only: tcpdump, HostAPd, WPA Supplicant and screen

## Installing required and optional dependencies

Below are instructions for installing the basic requirements to build
`aircrack-ng` for a number of operating systems.

**Note**: CMocka, tcpdump, screen, HostAPd and WPA Supplicant should not be dependencies when packaging Aircrack-ng.

### Linux

#### Arch Linux

    sudo pacman -Sy base-devel libnl openssl ethtool util-linux zlib libpcap sqlite pcre hwloc cmocka hostapd wpa_supplicant tcpdump screen iw usbutils pciutils

#### Debian/Ubuntu

    sudo apt-get install build-essential autoconf automake libtool pkg-config libnl-3-dev libnl-genl-3-dev libssl-dev ethtool shtool rfkill zlib1g-dev libpcap-dev libsqlite3-dev libpcre3-dev libhwloc-dev libcmocka-dev hostapd wpasupplicant tcpdump screen iw usbutils

#### Fedora

    sudo yum install libtool pkgconfig sqlite-devel autoconf automake openssl-devel libpcap-devel pcre-devel rfkill libnl3-devel gcc gcc-c++ ethtool hwloc-devel libcmocka-devel make file expect hostapd wpa_supplicant iw usbutils tcpdump screen zlib-devel

#### CentOS/RHEL 7

    sudo yum install epel-release
    sudo ./centos_autotools.sh
    # Remove older installation of automake/autoconf
    sudo yum remove autoconf automake
    sudo yum install sqlite-devel openssl-devel libpcap-devel pcre-devel rfkill libnl3-devel ethtool hwloc-devel libcmocka-devel make file expect hostapd wpa_supplicant iw usbutils tcpdump screen zlib-devel

**Note**: autoconf, automake, libtool, and pkgconfig in the repositories are too old. The script centos_autotools.sh automatically installs dependencies to compile then install the tools.

#### CentOS/RHEL 8

    sudo yum config-manager --set-enabled powertools
    sudo yum install epel-release
    sudo yum install libtool pkgconfig sqlite-devel autoconf automake openssl-devel libpcap-devel pcre-devel rfkill libnl3-devel gcc gcc-c++ ethtool hwloc-devel libcmocka-devel make file expect hostapd wpa_supplicant iw usbutils tcpdump screen zlib-devel

#### openSUSE

    sudo zypper install autoconf automake libtool pkg-config libnl3-devel libopenssl-1_1-devel zlib-devel libpcap-devel sqlite3-devel pcre-devel hwloc-devel libcmocka-devel hostapd wpa_supplicant tcpdump screen iw gcc-c++ gcc ethtool pciutils usbutils

#### Mageia

    sudo urpmi autoconf automake libtool pkgconfig libnl3-devel libopenssl-devel zlib-devel libpcap-devel sqlite3-devel pcre-devel hwloc-devel libcmocka-devel hostapd wpa_supplicant tcpdump screen iw gcc-c++ gcc make

#### Alpine

    sudo apk add gcc g++ make autoconf automake libtool libnl3-dev openssl-dev ethtool libpcap-dev cmocka-dev hostapd wpa_supplicant tcpdump screen iw pkgconf util-linux sqlite-dev pcre-dev linux-headers zlib-dev pciutils usbutils

**Note**: Community repository needs to be enabled for iw

#### Clear Linux

    sudo swupd bundle-add c-basic devpkg-openssl devpkg-libgcrypt devpkg-libnl devpkg-hwloc devpkg-libpcap devpkg-pcre devpkg-sqlite-autoconf ethtool wget network-basic software-testing sysadmin-basic wpa_supplicant

**Note**: hostapd must be compiled manually, it is not present in the repository

### BSD

#### FreeBSD

    pkg install pkgconf shtool libtool gcc9 automake autoconf pcre sqlite3 openssl gmake hwloc cmocka

#### DragonflyBSD

    pkg install pkgconf shtool libtool gcc8 automake autoconf pcre sqlite3 libgcrypt gmake cmocka

#### OpenBSD

    pkg_add pkgconf shtool libtool gcc automake autoconf pcre sqlite3 openssl gmake cmocka

### macOS

XCode, Xcode command line tools and HomeBrew are required.

    brew install autoconf automake libtool openssl shtool pkg-config hwloc pcre sqlite3 libpcap cmocka

### Windows

#### Cygwin

Cygwin requires the full path to the `setup.exe` utility, in order to
automate the installation of the necessary packages. In addition, it
requires the location of your installation, a path to the cached
packages download location, and a mirror URL.

An example of automatically installing all the dependencies
is as follows:

    c:\cygwin\setup-x86.exe -qnNdO -R C:/cygwin -s http://cygwin.mirror.constant.com -l C:/cygwin/var/cache/setup -P autoconf -P automake -P bison -P gcc-core -P gcc-g++ -P mingw-runtime -P mingw-binutils -P mingw-gcc-core -P mingw-gcc-g++ -P mingw-pthreads -P mingw-w32api -P libtool -P make -P python -P gettext-devel -P gettext -P intltool -P libiconv -P pkg-config -P git -P wget -P curl -P libpcre-devel -P libssl-devel -P libsqlite3-devel

#### MSYS2

    pacman -Sy autoconf automake-wrapper libtool msys2-w32api-headers msys2-w32api-runtime gcc pkg-config git python openssl-devel openssl libopenssl msys2-runtime-devel gcc binutils make pcre-devel libsqlite-devel

## Compiling

To build `aircrack-ng`, the Autotools build system is utilized. Autotools replaces
the older method of compilation.

**NOTE**: If utilizing a developer version, eg: one checked out from source control,
you will need to run a pre-`configure` script. The script to use is one of the
following: `autoreconf -i` or `env NOCONFIGURE=1 ./autogen.sh`.

First, `./configure` the project for building with the appropriate options specified
for your environment:

    ./configure <options>

**TIP**: If the above fails, please see above about developer source control versions.

Next, compile the project (respecting if `make` or `gmake` is needed):

 * Compilation:

    `make`

 * Compilation on *BSD or Solaris:

    `gmake`

Finally, the additional targets listed below may be of use in your environment:

 * Execute all unit testing:

    `make check`

 * Execute all integration testing (requires root):
 
    `make integration`

 * Installing:

    `make install`

 * Uninstall:

    `make uninstall`


###  `./configure` flags

When configuring, the following flags can be used and combined to adjust the suite
to your choosing:

* **with-airpcap=DIR**:  needed for supporting airpcap devices on windows (cygwin or msys2 only)
                Replace DIR above with the absolute location to the root of the
                extracted source code from the Airpcap CD or downloaded SDK available
                online. Required on Windows to build `besside-ng`, `besside-ng-crawler`, 
                `easside-ng`, `tkiptun-ng` and `wesside-ng` when building experimental tools.
                The developer pack (Compatible with version 4.1.1 and 4.1.3) can be downloaded at
                https://support.riverbed.com/content/support/software/steelcentral-npm/airpcap.html

* **with-experimental**: needed to compile `tkiptun-ng`, `easside-ng`, `buddy-ng`,
                    `buddy-ng-crawler`, `airventriloquist` and `wesside-ng`.
                    libpcap development package is also required to compile most of the tools.
                    If not present, not all experimental tools will be built.
                    On Cygwin, libpcap is not present and the Airpcap SDK replaces it.
                    See --with-airpcap option above.

* **with-ext-scripts**: needed to build `airoscript-ng`, `versuck-ng`, `airgraph-ng` and 
                   `airdrop-ng`. 
                   Note: Each script has its own dependencies.

* **with-gcrypt**:   Use libgcrypt crypto library instead of the default OpenSSL.
                And also use internal fast sha1 implementation (borrowed from GIT)
                Dependency (Debian): libgcrypt20-dev

* **with-duma**:	Compile with DUMA support. DUMA is a library to detect buffer overruns and under-runs.
            	Dependencies (debian): duma

* **disable-libnl**:  Set-up the project to be compiled without libnl (1 or 3). Linux option only.

* **without-opt**:  Do not enable stack protector (on GCC 4.9 and above).

* **enable-shared**:   Make OSdep a shared library.

* **disable-shared**: When combined with **enable-static**, it will statically compile Aircrack-ng.

* **with-avx512**:  On x86, add support for AVX512 instructions in aircrack-ng. Only use it when
                    the current CPU supports AVX512.

* **with-static-simd=<SIMD>**: Compile a single optimization in aircrack-ng binary. Useful when compiling
                    statically and/or for space-constrained devices. Valid SIMD options: x86-sse2,
                    x86-avx, x86-avx2, x86-avx512, ppc-altivec, ppc-power8, arm-neon, arm-asimd.
                    Must be used with --enable-static --disable-shared. When using those 2 options, the default
                    is to compile the generic optimization in the binary. --with-static-simd merely allows
                    to choose another one.

* **enable-maintainer-mode**: It is important to enable this flag when developing with Aircrack-ng. This flag enables additional compile warnings and safety features.

#### Examples:

  * Configure and compiling:

    ```
    ./configure --with-experimental
    make
    ```

  * Compiling with gcrypt:

    ```
    ./configure --with-gcrypt
    make
    ```

  * Installing:

    `make install`

  * Installing (strip binaries):
  
    `make install-strip`

  * Installing, with external scripts:

    ```
    ./configure --with-experimental --with-ext-scripts
    make
    make install
    ```

  * Testing (with sqlite, experimental and pcre)

    ```
    ./configure --with-experimental
    make
    make check
    ```

  * Compiling on OS X with macports (and all options):

    ```
    ./configure --with-experimental
    gmake
    ```

  * Compiling on macOS running on M1/AARCH64 and Homebrew:

    ```
    autoreconf -vif
    env CPPFLAGS="-Wno-deprecated-declarations" ./configure --with-experimental
    make
    make check
    ```

  * Compiling on OS X 10.10 with XCode 7.1 and Homebrew:

    ```
    env CC=gcc-4.9 CXX=g++-4.9 ./configure
    make
    make check
    ```

    *NOTE*: Older XCode ships with a version of LLVM that does not support CPU feature
    detection; which causes the `./configure` to fail. To work around this older LLVM,
    it is required that a different compile suite is used, such as GCC or a newer LLVM
    from Homebrew.

    If you wish to use OpenSSL from Homebrew, you may need to specify the location
    to its' installation. To figure out where OpenSSL lives, run:

    `brew --prefix openssl`

    Use the output above as the DIR for `--with-openssl=DIR` in the `./configure` line:

    ```
    env CC=gcc-4.9 CXX=g++-4.9 ./configure --with-openssl=DIR
    make
    make check
    ```

  * Compiling on FreeBSD with gcc9

    ```
    env CC=gcc9 CXX=g++9 MAKE=gmake ./configure
    gmake
    ```

  * Compiling on Cygwin with Airpcap (assuming Airpcap devpack is unpacked in Aircrack-ng directory)

    ```
    cp -vfp Airpcap_Devpack/bin/x86/airpcap.dll src
    cp -vfp Airpcap_Devpack/bin/x86/airpcap.dll src/aircrack-osdep
    cp -vfp Airpcap_Devpack/bin/x86/airpcap.dll src/aircrack-crypto
    cp -vfp Airpcap_Devpack/bin/x86/airpcap.dll src/aircrack-util
    dlltool -D Airpcap_Devpack/bin/x86/airpcap.dll -d build/airpcap.dll.def -l Airpcap_Devpack/bin/x86/libairpcap.dll.a
    autoreconf -i
    ./configure --with-experimental --with-airpcap=$(pwd)
    make
    ```

 * Compiling on DragonflyBSD with gcrypt using GCC 8

   ```
   autoreconf -i
   env CC=gcc8 CXX=g++8 MAKE=gmake ./configure --with-experimental --with-gcrypt
   gmake
   ```

 * Compiling on OpenBSD (with autoconf 2.69 and automake 1.16)

   ```
   export AUTOCONF_VERSION=2.69
   export AUTOMAKE_VERSION=1.16
   autoreconf -i
   env MAKE=gmake CC=cc CXX=c++ ./configure
   gmake
   ```

 * Compiling and debugging aircrack-ng

   ```
   export CFLAGS='-O0 -g'
   export CXXFLAGS='-O0 -g'
   ./configure --with-experimental --enable-maintainer-mode --without-opt
   make
   LD_LIBRARY_PATH=.libs gdb --args ./aircrack-ng [PARAMETERS]
   ```

# IDE development

## VS Code - devcontainers

A VS Code development environment is provided, as is, for rapid setup of a development environment. This additionally adds support for GitHub Codespaces.

### Requirements

The first requirement is a working [Docker Engine](https://docs.docker.com/engine/install/) environment.

Next, an installation of [VS Code](https://code.visualstudio.com/) with the following extension(s):

- [`Remote - Containers`](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) by Microsoft.

> The "Remote - Containers" extension will refuse to work with OSS Code.

### Usage

1. Clone this repository to your working folder:
```
$ git clone --recursive https://github.com/aircrack-ng/aircrack-ng.git
$ cd aircrack-ng
```
2. After cloning this repository, open the folder inside VS Code.
```
$ code .
```
> IMPORTANT: You should answer "Yes", if it asks if the folder should be opened inside a remote container. If it does not ask, then press `Ctrl+Shift+P` and type `open in container`. This should bring up the correct command, for which pressing enter will run said command.

3. A number of warnings might appear about a missing `compile_commands.json` file. These are safe to ignore for a moment, as this file is automatically generated after the initial compilation.
4. Now build the entire project by pressing `Ctrl+R` and selecting `Build Full` from the pop-up menu that appears.
5. VS Code should detect the `compile_commands.json` file and ask if it should be used; selecting "Yes, always" will complete the initial setup of a fully working IDE.
> IMPORTANT: If it doesn't detect the file, pressing `Ctrl+Shift+P` and typing `reload window` will bring up the selection to fully reload the environment.
6. At this point, nearly all features of VS Code will function; from Intellisense, auto-completion, live documentation, to code formatting. Additionally, there are pre-configured tasks for builds and tests, as well as an example GDB/LLDB configuration for debugging `aircrack-ng`.

# Packaging

Automatic detection of CPU optimization is done at run time. This behavior
**is** desirable when packaging Aircrack-ng (for a Linux or other distribution.)

Also, in some cases it may be desired to provide your own flags completely and
not having the suite auto-detect a number of optimizations. To do this, add
the additional flag `--without-opt` to the `./configure` line:

`./configure --without-opt`

# Using pre-compiled binaries

## Linux/BSD

Aircrack-ng is available in most distributions repositories. However, it is not always up to date.

We provide up to date versions via PackageCloud for a number of Linux distributions:

- development (each commit in this repo): https://packagecloud.io/aircrack-ng/git
- stable releases: https://packagecloud.io/aircrack-ng/release

## Windows
 * Install the appropriate "monitor" driver for your card; standard drivers don't work for capturing data.
 * Aircrack-ng suite is command line tools. So, you have to open a command-line
   `Start menu -> Run... -> cmd.exe` then use them
 * Run the executables without any parameters to have help

# Continuous integration

- Linux CI (GitHub actions): https://github.com/aircrack-ng/aircrack-ng/actions/workflows/linux.yml
- Windows CI (Github actions): https://github.com/aircrack-ng/aircrack-ng/actions/workflows/windows.yml
- macOS CI (GitHub actions): https://github.com/aircrack-ng/aircrack-ng/actions/workflows/macos.yml
- Code style and consistency (GitHub actions): https://github.com/aircrack-ng/aircrack-ng/actions/workflows/style.yml
- PVS-Studio static analysis (GitHub actions): https://github.com/aircrack-ng/aircrack-ng/actions/workflows/pvs-studio.yml
- Coverity Scan static analysis: https://scan.coverity.com/projects/aircrack-ng

## Buildbots

URL: https://buildbot.aircrack-ng.org/

Linux buildbots:
- CentOS
- AArch64
- Kali Linux
- Armel Kali Linux
- Armhf Kali Linux
- Alpine Linux

BSD buildbots:
- OpenBSD
- FreeBSD
- NetBSD
- DragonflyBSD

# Documentation

Some more information is present in the [README](README) file.

Documentation, tutorials, ... can be found on https://aircrack-ng.org

Support is available in the [forum](https://forum.aircrack-ng.org) and on IRC (in #aircrack-ng on Libera Chat).

Every tool has its own manpage. For aircrack-ng, `man aircrack-ng`

# Infrastructure sponsors
