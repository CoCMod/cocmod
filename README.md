# Clash of Clans Mod

This repository documents the steps taken thus far to get iMod to work under Clash of Clans 8.551.2

## Overview

The following steps must be taken to create a viable build of a modified Clash of Clans of application. Most of which is related to subverting Apple's security and code signing prequisites.

1. Decrypt the 32bit ARM slice of the Clash of Clans binary (via [dumpdecrypted](https://github.com/stefanesser/dumpdecrypted))
2. Retrieve the ARMV7 architecture via the [lipo](http://www.manpages.info/macosx/lipo.1.html) command 
3. Use the install_dylib binary (located in this repository) to insert the iMod.dylib into the Clash of Clans load path
4. Resign the binary using the [ldid](http://iphonedevwiki.net/index.php/Ldid) command
5. Replace the existing FAT Clash of Clans binary
6. Update the Info.plist
   1. Bundle Identifier
   2. App URLs
   3. Bundle Name
7. Generate a new IPA file
    1. An IPA file is just a zip archive with well-defined structure
8. Deploy via [Cydia Impactor](cydiaimpactor.com)

## Installing

The latest IPA is available in this repository. However, it is not currently functional and will crash on launch. If you are interested in reviewing the crash logs to determine next steps, feel free.

Use [Cydia Impactor](cydiaimpactor.com) to install the IPA

## Coming Soon

1. Automatic build process via Rake
2. More in-depth documentation

## Known Issues

Currently, iMod is _not_ compatible with the new Clash of Clans release. 

iMod attempts to retrieve or access a file that no longer exists in the original location (NSKeyedArchiver arror):

```
[NSKeyedUnarchiver initForReadingWithData:]: data is NULL
```

## Building

### Decrypting Clash Binary

Checkout and compile dumpdecrypted

```bash
make
scp dumpdecrypted.dylib ssh root@DEVICEIP:/usr/lib
```

```bash
ssh root@DEVICEIP
chmod +x /usr/lib/dumpdecrypted.dylib
```

```bash
DYLD_INSERT_LIBRARIES=/usr/lib/dumpdecrypted.dylib /var/mobile/Containers/Bundle/Application/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/Clash\ of\ Clans.app/Clash\ of\ Clans
```

Where the _XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX_ is the bundle name of the Clash of Clans install. Executing that command will produce a decrypted MACH-O file. You _must_ do this on a 32bit device since iMod has not been compiled for arm64.

### Retrieve ARMV7 Architecture Slice

```bash
lipo -thin armv7 Clash\ of\ Clans.decrypted -output Clash\ of\ Clans.armv7
```

### Insert iMOD DYLIB

```bash
insert_dylib @executable_path/iModGame/iModGame.dylib Clash\ of\ Clans.armv7
```

### Replace existing binary

```
cd <PATH>/Clash\ of\ Clans.app
cp <PATH>/Clash\ of\ Clans.armv7 Clash\ of\ Clans
ldid -S Clash\ of\ Clans
```

### Update Info.plist

COMING SOON

### Generate new IPA FIle

```
zip -r CoCMod.ipa Payload iTunesArkwork
```

Where the Payload contains the Clash of Clans.app
