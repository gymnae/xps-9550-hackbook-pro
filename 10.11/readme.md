[ATTACH=full]220324[/ATTACH]
[SIZE=5]Overview[/SIZE]
The XPS 15 9550 is a Skylake Laptop with decent Hardware for running a Hackintosh without too much fiddling.
Details of the system: http://www.dell.com/us/p/xps-15-9550-laptop/pd?oc=dncwx1631h&model_id=xps-15-9550-laptop

Please keep the discussion as close to El Capitan as possible. For Sierra specific questions please check the Sierra subforum.
Are you a (potential) Sierra user? @wmchris has you covered: https://www.tonymacx86.com/threads/guide-dell-xps-15-9550-sierra-10-12-2-tutorial.210368/

Thanks to the entire Hackintosh community, in particular to these members:
- rehabman
- tdmsn
- reece394
- pikeralpha
- goodwin_c
- daved314

[SIZE=4]Changes to the guide[/SIZE]
Re-patched for new BIOS by Dell, which changed a lot ACPI wise - changes to be found in repo & Sierra guide linked
Two really nice additions: Refined brightness patch AND retain Speed Shift after resume from sleep
Added DSDT patch for retaining brightness upon resume
Slightly more clarity on the SSDTs needed for usbinjectall and nullethernet
Added Hardware calibrated ICC profile for 4k screen
Added HDMI output fix and deleted iMac17,1 SMBIOS & added github repo
A solid mix of changes before goodwin_c releases his Sierra guide in the coming days / weeks to get this guide back on par with developments:
Native AppleHDA patching, good riddance VoodooHDA
NVMe patching to get rid off NVMeGeneric
Enable Speed Shift for fancy Skylake power management
Added USBInjectAll.kext for using the internal Webcam &
Access to App Store and iMessage via NullEthernet.kext
[SPOILER="Old changelog"]
[*]Sadly, due to my lack of time I couldn't keep up with the changes and improvements found in the wealth of the thread to this guide. With Sierra out and the recent introduction of MBP's nearly identical to this 2015 device (ah Apple, so sad) (minus our better discrete GPU) a Sierra guide will be done - but by someone more capable. I will update this guide with the latest changes to finalize it once the Sierra guide is out and I can incorporate the lessons learned since writing this up.
[*]Coming: Changes and additions from the collaboration and notes for the new 2.10 BIOS from July 5th
[*]10.11.5 update & alternative IOKIT patching
[*]More info on injecting the ID for the Intel IGPU based in i5 or i7 & added minimal install Clover example folder
[*]Fixed my iMessage problems with a proper MLB value and added warning about NVMeGeneric Kernel Panics
[*]Added warning about DMVT memory pre-alloc and noted patch required

[*]Added tdmsn's edits to SMBIOS MacBook9,1 and Nvidia GPU turn off, some snippet for Audio

[*]Initial writeup of guide for 10.11.4[/SPOILER]
[SIZE=4]Status of the system[/SIZE]

Working
97% of the system
Not working
SD card reader
TB3 and USB 3.1 due to macOS 10.11 limitation - solved in macOS 10.12

[SIZE=5]1. Prerequisites:[/SIZE]
Please read and follow the FAQ by rehabman. Also the SSDT/DSDT patching guide. Read carefully, most of my problems came from me not reading thoroughly.
After that we can continue.

[SIZE=4]1.1. Bios[/SIZE]
The BIOS should be set according to rehabman's Clover install guide:

[QUOTE]In order to boot the Clover from the USB, you should visit your BIOS settings:
- "VT-d" (virtualization for directed i/o) should be disabled if possible (the config.plist includes dart=0 in case you can't do this)
- "DEP" (data execution prevention) should be enabled for OS X
- "secure boot " should be disabled
- "legacy boot" optional
- "CSM" (compatibility support module) enabled or disabled (varies)
- "boot from USB" or "boot from external" enabled

Note: If you get a "garbled" screen when booting the installer in UEFI mode, enable legacy boot and/or CSM in BIOS (but still boot UEFI). Enabling legacy boot/CSM generally tends to clear that problem.[/QUOTE]

In my case I left VT-d and Fastboot as they were. Also, update your 9550 to the latest BIOS.

Don't forget to set mode to "AHCI" in the sub-menu "SATA Operation" of "System Configuration". It's mandatory.

[SIZE=3](Old) Warning for 4k / UHD screen owners:[/SIZE]
Do not use the EFI shell to to edit the DVMT Pre-allocation for getting your screen to work. This bricked a user's machine already. Instead, make use of a patch explained in a later section of this guide. All you need to do is follow the instruction in this guide. If you do not know about DVMT Pre-allocation, do not worry. You not need it. So in short: Do not use UEFI shell to edit DVMT allocation.


[SIZE=4]1.2 Prepare the USB[/SIZE]
Simply follow rehabman's install guide referenced earlier for a USB install. For this you need a real Mac or a Hack with access to the Apple Store for downloading 10.11.4 or newer - >= .4 is necessary for initial Skylake Support.  If you download freshly from the Apple Store, you'll get 10.11.5 or newer

During the preparation of the USB install it's best to go for a fail-safe clover configuration. A streamlined, all patches active configuration comes after your system said Hello world and is ready for usage.

In short, please use this plist by rehabman for the initial setup on your USB stick. This plist includes important patches, some of which we'll activate after the initial installation.

In this plist you need to adjust a couple of things:

Into "Devices" section:
[code]<key>FakeID</key>
        <dict>
            <key>IntelGFX</key>
            <string>0x12345678</string>
        </dict>[/code]
This will turn off the acceleration and thus the full driver, but it's enough to install and a safe bet. For the 4k screen to work, you'll also need to make a patch post-install, see below. Only post install! Only after making the patch should continue to enable QE/CI

Into "Graphics" section:
For injecting the Intel IGPU into OS X, you need to give Clover the correct ID.

For i7 based systems
[code]    <key>ig-platform-id</key>
        <string>0x191b0000</string>[/code]

For i5 based systems
[code]    <key>ig-platform-id</key>
        <string>0x19160000</string>[/code]

The rest of the config.plist can stay as rehabman pre-configured it. Please don't make too many edits to your config.plist before finalising your install.

[SIZE=4]1.3 Kernel Extensions for Install[/SIZE]
You may need extra kexts, apart from the mandatory FakeSMC and VoodooPS2 (as per rehabman's install guide):
HackrNVMeFamily-10_11_6.kext - for seeing and installing on a stock SSD if you have a M2 NVMe SDD! It's attached to this guide and valid for 10.11.6 - see below in 3.7 if you are installing an earlier or later version of MacOS.
Recommended escalation of getting your drive to cooperate:
Use the HackrNVMeFamily kext
If the Installer doesn't see your SSD with this kext, then it might be missing the patches for your device. In this case you have to use NVMeGeneric for install. Please report if you have to do so, maybe your drive can be "hacked" post-install. Of course, delete HackrNVMeFamily
Should you still face problems, see this post.

Please refrain from using other kexts during install. Go with a bare minimum. All you want is to get the system on your Laptop and boot.

Please see 3.7 of the post install section to update the included HackrNVMeFamily-10_11_6.kext to newer OS versions or even Sierra

[SIZE=5]2. Installation[/SIZE]
Installation is pretty much 100% in alignment with rehabman's guide

Just make sure, again, to follow the installation guide

I split up the drive into two partitions to prepare the second one for Windows later. So I left the second one just untouched by formatting it HFS+ which was later overwritten by the Windows 10 install routines. But that is just if you also want to use the silenced beast that is the GTX 960M for gaming outside of serious Mac business.

How to get MacOS booting again after installing Windows:
When installing Windows after having installed Clover, it will install its own bootloader into your EFI partition. To get back to boot macOS, you need to have the USB key from the install handy, so you can add Clover's UEFI boot entries. So don't panic :)

[SIZE=5]3. Post install[/SIZE]
After your install went smooth and you installed Clover to your SSD as per guide, it's time for the real work.

It's also a good idea to keep the USB stick handy and not fiddle with it's config. It was good enough for a install, it's good enough to start your machine with and fix the errors your *.dsl patching and config.plist adjustments brought along.

[SIZE=4]3.1 DSDT/SSDT Patching[/SIZE]
[SPOILER="Notes on Dell's BIOS and Firmware updates"]The BIOS of this Dell is still a work in progress the fans run too often, Windows 10 suffers from BSODs when browsing etc. The latest version as of this writing (November 2016) has a PWN flickering bug at lowest brightness setting and deleted the very lowest brightness setting. There also has been a swell of Firmware updates for other components.

After updating firmware or BIOS, re-patching might become necessary. So please try to restrain from updating when your system runs stable and you are happy just for the sake of updating.

Thus keep the scripts and patching info at hand, because you will have to re-patch when necessary updates come through.
With this out of the way, let's learn about the patches.
[/SPOILER]

Patching DSDTs and SSDTs will enable the following:
Turn off the NVIDIA GPU for OS X - this is a must - it saves a lot of battery and the discrete GPU won't be utilized by MacOS anyway
Backlight control, important for proper sleep and resume
Fn keys for Backlight control via Fn keys
Audio, see below for the special section pertaining how to enable our codec
rehabman, again, provides all info needed for patching these files. His master guide is a "must be open at all time"

[SIZE=4]3.1.1 Extract[/SIZE]
When in the Clover boot menu, press F4 to extract vanilla DSDT and SSDTs from the system and save them in your Clover ACPI folder. These files shall be the base for all the patching following.

[SIZE=4]3.1.2 Prepare[/SIZE]
For all DSDTs and SSDTs, please remove or rename [code]_DSM[/code] methods via rehabman's patch repo, which is now neatly waiting in your MacIASL if you read the guide by rehabman. If I haven't mentioned before, read his guide before continuing to avoid mistakes and unnecessary questions.

After you extracted your DSDT and SSDTs, you need to descramble them as guided:

Important is to use a refs.txt such as this:
[code]External(MDBG, MethodObj, 1)
External(_GPE.MMTB, MethodObj, 0)
External(_SB_.PCI0.LPCB.H_EC.ECWT, MethodObj, 2)
External(_SB_.PCI0.LPCB.H_EC.ECRD, MethodObj, 1)
External(_SB_.PCI0.PEG0.PEGP.SGPO, MethodObj, 2)
External(_SB.PCI0.GFX0.DD02._BCM, MethodObj, 1)
External(_SB.PCI0.SAT0.SDSM, MethodObj, 4)
External(_SB.PCI0.SAT1.SDSM, MethodObj, 4)
External(_GPE.VHOV, MethodObj, 3)[/code]

And the following command:
[code] iasl -da -dl -fe refs.txt *.aml[/code]

Now you have a set of descrambled *.dsl files and you do want to make a backup of them.

When using patched DSDT and SSDTs it's necessary to add the following to your config.plist in "SSDT"
[code]<key>DropOem</key>
            <true/>[/code]

The preparation work is done, let's patch!

For starters, you can check the DSDT and SSDTs attached in the archive. But you should extract yourself and patch yourself. Who knows what Firmware and Bios is driving your 9550.

The first patch you should apply to your DSDT is "Rename _DSM methods to XDSM" - then continue. rehabman's guide also gives hints at usually good and unproblematic patches. Try it yourself - rather don't patch too much, try to boot and see if you get the desired effect after every patch.

[SIZE=4]3.2.1 Backlight control[/SIZE]
Patching guide for backlight control. All you need is the "OS Check Fix (Windows 8)" and a kext, see below.

In addition to this, you need to follow @tdmsn's findings for adjusting your DSDT for getting our dedicated brightness keys to work. One of the following patches need to be added to your DSDT via MaciASL's patch window:

For VoodoPS2Trackpad:
[code]into method label BRT6 replace_content
begin
If (Arg0 == 1) { Notify (^^LPCB.PS2K, 0x10) }\n
If (Arg0 == 2) { Notify (^^LPCB.PS2K, 0x20) }\n
end;[/code]

If you opt for the ApplePS2SmartTouchPad kext:
[code]into method label BRT6 replace_content
begin
If (Arg0 == 1) { Notify (\_SB.PCI0.LPCB.PS2K, 0x0406) }\n
If (Arg0 == 2) { Notify (\_SB.PCI0.LPCB.PS2K, 0x0405) }\n
end;[/code]

I opted for ApplePS2SmartTouchPad.kext - I found it to offer better Palm rejection while also supporting more gestures. Version 4.x works just fine.

Retain brightness value upon resume from sleep and benefit from super fine brightness steps
This tip is courtesy of [USER=12214]@dpassmor[/USER]:
If installed, remove 'IntelBacklight.kext' from S/L/E
If patched, remove brightness fixes from DSDT
Remove the existing PNLF device from the DSDT
Compile and install the SSDT from this post to the 'PATCHED' folder of your Clover installation - add it to your named list of SSDTs in config.plist, if you are using a named list
Create a 'AppleBacklightInjector' kext as mentioned in the post above and install it in S/L/E
Rebuild kextcache
[code]sudo kextcache -system-prelinked-kernel
sudo kextcache -system-caches[/code]

[SIZE=4]3.2.2 Turn off NVIDIA[/SIZE]
This video plus rehabman's guide helped me.
[MEDIA=youtube]KBOZQL3uBE4[/MEDIA]

I patched most of the SSDTs and the DSDT, but this may not be necessary:

[USER=1441867]@tdmsn[/USER] patched mostly the DSDT only and turnf the discrete GPU off as well. His method is in this post

[SIZE=4]3.2.3 Rename iGPU[/SIZE]
There are patches and a guide by rehabman

Most essential for me to get it working were:
"Rename GFX0 to IGPU"
"Cleanup/Fix Errors (SSDT)"
But it always depends on your SSDTs. Just be sure to follow the guide, especially making sure you treat every SSDT with the rename.

[SIZE=5]3.3 SSDT.aml via pikeralpha[/SIZE]
Needed for native Power Management. Please refer, to once again, a great guide by rehabman.
You have to chose between this method for power management and the new fancy Speed Shift method mentioned below. Speed Shift ignores SSDT and OS-level settings.

[SIZE=5]3.4. More kexts[/SIZE]
Besides the kexts you already had during install, you could install a couple of more into the "Other" folder of Clover:
BrcmFirmwareData - For Bluetooth
BrcmPatchRAM2 - For Blueetooth
VoodooHDA - Works, but not recommended
ACPIBatteryManager - for displaying the battery and its status correctly
FakePCIID - for enabling the kext injector below
FakePCIID_Intel_HD_Graphics - for the Intel iGPU
NullEthernet & SSDT-renameme - for getting an en0 device recognized as built-in (App Store, iMessage etc)
USBInjectAll - for using the Webcam
some say using this kext is too much and injecting the needed USB ports via a patch is better, your choice
If using this "hack", you also need the file SSDT-UIC-ALL.ssdt (see my repo for an example)
ApplePS2SmartTouchPad - a good Touchpad and Keyboard driver
As you read above, one injector should be installed to S/L/E by now:
AppleBacklightInjector
[SIZE=5]3.5 config.plist additions[/SIZE]
To make it simple and for starters, you can be inspired by the config.plist from the achive attached.
Edits to this file will yield the following:
Full QE/CI by removing the FakeID from the install
Fully activating the internal GPU
Adding an SMBIOS definition closer to the real system
BT handsoff / 5Ghz
Drop SSDT Oem - see section above

[SIZE=4]3.5.1 5ghz patch[/SIZE]
Enter this into your config.plist
[code]<dict>
                <key>Comment</key>
                <string>10.11-BCM94352-5GHz-US-FCC-dv</string>
                <key>Disabled</key>
                <false/>
                <key>Find</key>
                <data>
                QYP8/3QsSA==
                </data>
                <key>Name</key>
                <string>AirPortBrcm4360</string>
                <key>Replace</key>
                <data>
                ZscGVVPrKw==
                </data>
            </dict>[/code]

[SIZE=4]3.5.2 Handoff patch[/SIZE]
[code]<dict>
                <key>Comment</key>
                <string>10.11.dp1+ BT4LE-Handoff-Hotspot, credit RehabMan based on Dokterdok original</string>
                <key>Disabled</key>
                <false/>
                <key>Find</key>
                <data>
                SIX/dEdIiwc=
                </data>
                <key>Name</key>
                <string>IOBluetoothFamily</string>
                <key>Replace</key>
                <data>
                Qb4PAAAA60Q=
                </data>
            </dict>[/code]

[SIZE=4]3.5.3 Fix Shutdown[/SIZE]
The system will reboot when you tell it to shutdown. To fix this, add this to your config.plist in "Fixes"
[code]<key>FixShutdown_0004</key>
                <true/>[/code]

[SIZE=4]3.5.4 Fix HDMI output[/SIZE]
[code]<dict>
                <key>Comment</key>
                <string>10.11-SKL-1912000-4_displays</string>
                <key>Find</key>
                <data>
                AQMDAw==
                </data>
                <key>Name</key>
                <string>AppleIntelSKLGraphicsFramebuffer</string>
                <key>Replace</key>
                <data>
                AQMEAw==
                </data>
            </dict>
<dict>
                <key>Comment</key>
                <string>Fix HDMI output</string>
                <key>Find</key
<string>3e4d61632d423830394333373537444139424238443c2f6b65793e0a090909093c737472696e673e436f6e666967323c2f737472696e673e0a09</string>
                <key>InfoPlistPatch</key>
                <true/>
                <key>Name</key>
                <string>AppleGraphicsDevicePolicy</string>
                <key>Replace</key
<string>3e4d61632d423830394333373537444139424238443c2f6b65793e0a090909093c737472696e673e6e6f6e653c2f737472696e673e0a09090909</string>
            </dict>[/code]

[SIZE=4]3.5.5 SMBIOS[/SIZE]
[USER=1441867]@tdmsn[/USER] is using an MacBook9,1 SMBIOS and has no problem with iCloud, App Store etc.:
[code]<key>SMBIOS</key>
    <dict>
        <key>BiosReleaseDate</key>
        <string>01/18/16</string>
        <key>BiosVendor</key>
        <string>Apple Inc.</string>
        <key>BiosVersion</key>
        <string>MB91.88Z.0154.B00.1603041656</string>
        <key>Board-ID</key>
        <string>Mac-9AE82516C7C6B903</string>
        <key>BoardManufacturer</key>
        <string>Apple Inc.</string>
        <key>BoardType</key>
        <integer>10</integer>
        <key>ChassisAssetTag</key>
        <string>MacBook-Aluminum</string>
        <key>ChassisManufacturer</key>
        <string>Apple Inc.</string>
        <key>ChassisType</key>
        <string>08</string>
        <key>Family</key>
        <string>MacBook Pro</string>
        <key>Manufacturer</key>
        <string>Apple Inc.</string>
        <key>Memory</key>
        <dict>
            <key>Channels</key>
            <integer>2</integer>
            <key>Modules</key>
            <array>
                <dict>
                    <key>Frequency</key>
                    <string>2133</string>
                    <key>Part</key>
                    <string>M471A1G4EB0-CPB</string>
                    <key>Serial</key>
                    <string>1200B13D</string>
                    <key>Size</key>
                    <string>8192</string>
                    <key>Slot</key>
                    <string>2</string>
                    <key>Type</key>
                    <string>DDR4</string>
                    <key>Vendor</key>
                    <string>Samsung</string>
                </dict>
                <dict>
                    <key>Frequency</key>
                    <string>2133</string>
                    <key>Part</key>
                    <string>M471A2T3DB0-CPB</string>
                    <key>Serial</key>
                    <string>1200B1B4</string>
                    <key>Size</key>
                    <string>8192</string>
                    <key>Slot</key>
                    <string>0</string>
                    <key>Type</key>
                    <string>DDR4</string>
                    <key>Vendor</key>
                    <string>Samsung</string>
                </dict>
            </array>
            <key>SlotCount</key>
            <integer>4</integer>
        </dict>
        <key>Mobile</key>
        <true/>
        <key>ProductName</key>
        <string>MacBook9,1</string>
        <key>SerialNumber</key>
        <string>C02K***FD56</string>
        <key>Trust</key>
        <true/>
        <key>Version</key>
        <string>1.0</string>
    </dict>[/code]

Obviously, you must find your own Serial No. - If you take the SMBIOS posted here and simply copy and paste them, they WILL not work

[SIZE=5]3.6 Full QE/CI, 4k and fix memory allocation[/SIZE]
Thanks to 10.11.4+ this is super simple. Just take out the Fake-ID and ensure you kept the InjectedID as in the beginning of the guide. If you employ the Backlight Fix and IntelBacklight.kext - which you should - you'll have a super smooth boot-up experience, too.

[SIZE=3]Additional steps for 4k / UHD screens:[/SIZE]
If you have a 4k screen, you must make two additional changes, courtesy of the guide by "the-darkvoid" for the Dell 9350 and rehabman:

Open a Terminal and run these commands:
[code]# Run the following command in order to patch IOKit in order to disable CheckTimingWithRange
sudo perl -i.bak -pe 's|\xB8\x01\x00\x00\x00\xF6\xC1\x01\x0F\x85|\x33\xC0\x90\x90\x90\x90\x90\x90\x90\xE9|sg' /System/Library/Frameworks/IOKit.framework/Versions/Current/IOKit
sudo codesign -f -s - /System/Library/Frameworks/IOKit.framework/Versions/Current/IOKit[/code]

Only after these changes will 4k work. Which you want, if you paid $$$ for 100% AdobeRGB


[SIZE=5]3.7 iMessage, App Store, iCloud etc. [/SIZE]
For proper functionality you need to generate MLB and SMUUID. As usually, the guide from rehabman covers that in depth. I was successful with the following steps:
Generate a proper Serial No not in use yet by others. Either through Clover configurator for supported System and its SMBIOS generator, or by hand
Make sure you don't have a MLB or SmUUID value set
You can take the Board-ID and Bios from the examples in this guide for the system definitions provided
Reboot after setting Serial number, Board-ID and Bios
Use Clover Configurator's Rt variable generator for getting the MLB, SmUUID value. ROM is usually set correctly automatically.
Save the details given by the configurator into SMBIOS fields
Reboot and try iCloud, Message etc.
I also found that the aforementioned NullEthernet.kext enables a reliable connection to these services. Please follow the instructions when installing.
As explained by rehabman, you also need to place ssdt-rmne.ssdt with a custom MAC address and edit the ssdt into your config.plist

For advanced help, please see this thread


[SIZE=5]3.8 Audio[/SIZE]
Since I last wrote about Audio, things have improved a lot. No more Voodoo, now let's patch native via injection, yeah.
You need two kexts and the SSDT-ALC298.aml attached to this post
Place CodecCommander and AppleALC into your Clover kexts folder
Place the afromentioned ALC298.aml int your ACPI/patched folder - don't forget to adjust your config.plist if you use a named SSDT list
Inject Audio ID 13 via config.plist
If you previously committed Audio patches, like injecting Audio ID through DSDT or even renaming the Audio device (I did that, thus finding the device was a bit harder), then you need to adjust the Audio ID in your DSDT as well

[code]into method label _DSM parent_label HDEF remove_entry;
into device label HDEF insert
begin
Method (_DSM, 4, NotSerialized)\n
{\n
    If (LEqual (Arg2, Zero)) { Return (Buffer() { 0x03 } ) }\n
    Return (Package()\n
    {\n
        "layout-id", Buffer() { 13, 0x00, 0x00, 0x00 },\n
        "hda-gfx", Buffer() { "onboard-1" },\n
        "PinConfigurations", Buffer() { },\n
        //"MaximumBootBeepVolume", 77,\n
    })\n
}\n
end;[/code]

[SIZE=5]3.9 Speed Shift[/SIZE]
Skylake brings a new, fancy method of managing your C- and P-States of your CPU called Speed Shift. This means quicker throttling up and down, can mean higher power or longer battery, depending on your needs. Bare in Mind, this is a pretty new development in the Hackintosh world, so use this as an alternative to generating states via the guide linked in 3.4.
If you are interested (I'm happily using it, it keeps the fan on lower settings), there are two ways:

1. Enable HWP in Clover:
[code]<key>CPU</key>
    <dict>
        <key>HWPEnable</key>
        <true/>
    </dict>[/code]

2. Use the dedicated kext by fellow board member goodwin_c:
[Release] HWP (Intel Speed Shift on Skylake and up) Enabler Kext
The kext allows for fine granular control and thus adjusting the power profile to your liking.

Some important notes when using HWP:
[QUOTE]- My changes for HWP support in Clover are already in repo and will be included into next build ;) https://sourceforge.net/p/cloverefiboot/code/3879/
- Fully rewrote my kext. I like it now! You can got it here https://github.com/goodwin/HWPEnable
- Normal support of HWP needs not only CPU configuration, but also frequencyVectors with HWP flag. Currently in El Capitan this is only MacBook9,1. So who want to have best possible CPU performance with incredible battery life - change your smbios to MacBook9,1. Also, for those who has generated ssdt.aml - delete it and forget, we don't need it anymore bcz HWP will handle all power management! Yarrr[/QUOTE]

Yes, change your SMBIOS to MacBook9,1 - with Sierra this could be MacBookPro13,1 as well, we'll have to see.

Keep Speed Shift after sleep
To retain Speed Shift after resume from sleep, you need to follow a few simple steps outlined in this post. Otherwise, Speed Shift won't work after your system slept. Kinda pointless, so follow the simple steps.

Remember: It's either Speed Shift or SSDT.aml patching via Pike R. Alpha for Power Management, not both.


[SIZE=5]3.10 Native NVMe[/SIZE]
In the pasts months the Hackintosh world has evolved even further, also again thanks to the work of the likes of rehabman and pikeralpha. A new method has been found to directly patch NVMe kexts by Apple to avoid NVMeGeneric

NVMe patching
[QUOTE]
This script can be used to create patched IONVMeFamily.kext for non-Apple NVMe SSDs, such as the Samsung 950 Pro NVMe.

The scripts implement the patches created by Pike R. Alpha and Mork vom Ork at Pike's blog.
[/QUOTE]

In short, could run this after you've installed macOS 10.11.
[code]mkdir ~/Projects && cd Projects
git clone https://github.com/RehabMan/patch-nvme.git patch-nvme.git
cd patch-nvme.git
./patch_nvme.sh 10_11_6[/code]

This will yield in a kext you can install or add to your Clover kexts.
This method could also be used to create a kext for NVMe patching before updating to 10.12 - you need to adjust the target platform in the command line accordingly.

[SIZE=5]4. Updating[/SIZE]
[SIZE=4]4.1 10.11.6 & Security updates[/SIZE]
Updating to 10.11.6 worked for me via the App Store (see 3.3.6 for getting the App Store to work). Here's the steps I took:
Make sure that the App Store is working by providing a proper SMBIOS
Download the Update and let it install itself, it will reboot when it's done to the Clover boot menu
With FHD, you're done at this point, apart from re-patching or re-installing kexts in S/L/E
Additional steps for 4k/UHD
After the initial installation restart, don't boot directly boot OS X, instead enter Clover Options during OS selection after boot
In the Clover boot menu, open the "Graphics injector menu" and inject a fake ID for the Intel IGPU, much like during install, for example [code]0x12345678[/code] - this edit is non-permanent and just valid for the coming start of OS X
Now boot macOS and re-patch IOKIT
[code]# Run the following command in order to patch IOKit in order to disable CheckTimingWithRange
sudo perl -i.bak -pe 's|\xB8\x01\x00\x00\x00\xF6\xC1\x01\x0F\x85|\x33\xC0\x90\x90\x90\x90\x90\x90\x90\xE9|sg' /System/Library/Frameworks/IOKit.framework/Versions/Current/IOKit
sudo codesign -f -s - /System/Library/Frameworks/IOKit.framework/Versions/Current/IOKit[/code]
After patching, simply reboot and you're golden - you may need to reinstall .kexts or kext patches depending on your setup
[SIZE=5]ToDo[/SIZE]
I probably forgot important things, so this guide will be updated accordingly.
AppleHDA patching instead of Voodoo Audio
Hopefully better palm rejection for the Touchpad - solved by an alternative TouchPad driver
Replacing NVMeGeneric with a more stable method
[SIZE=5]Repo[/SIZE]

A git of the current kexts, cofnigs & ACPI files I use can be found here. As always, be careful when using ACPI files from other systems.

[SIZE=5]Files attached[/SIZE]
All the files attached are tuned to i7 systems. You need to adjust the injected ID pertaining your CPU (see above for details) as well as decide on using NVMeGeneric.

Clover.zip: This zip file provided should only be a starting point to discover how a complete system is configured. It's based on my running i7 setup - It's not a good idea to deliver a running setup, as it maybe not up to date and not optimized for your system. Instead, use this guide and rich thread to create your streamlined, optimized setup. Everything you need is there.

minimal_for_install_example.zip: This is the actually setup I used when installing the system. It's minimal, without many kexts and config.plist edits

SSDT-ALC298.aml: The verb to be used in combination with CodecCommander and AppleALC to relish native audio support

HackrNVMeFamily-10_11_6.kext - Required to install and use MacOS on 9550s fitted with NVMe M.2 drives

Display #1 2016-11-13 14-13 2.2 F-S XYZLUT+MTX - ICC profile for 4k screen calibrated with Spyder4Pro colorimeter and DisplayCAL. Every panel is different, so don't expect too much precision, but this profile works great for sRGB and AdobeRGB

dellxps15icon.png - An icon to brand your Desktop :) Apply with LiteIcon or manually

Please take care when using files contained, especially ACPI related ones.[/size]