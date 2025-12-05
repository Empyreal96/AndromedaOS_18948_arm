# Guide for building AndromedaOS 10.0.18948.1001 for Talkman devices

Note: There will be some things you'll be required to do, mainly extract any *.mbn firmware from your 950 running a production Windows 10 Mobile version.
Other things will be mentioned below.

**!! You will need to obtain a Windows Assessment and Deployment Kit that's version is as close to 18948 as possible. (19041 ADK doesn't work for this build) !!**

Requirements:
- The [fixed package set](https://github.com/Empyreal96/AndromedaOS_18948_arm/releases/tag/v1)
- Windows ADK.
- Visual Studio 2019/2022 to compile the older build of WPInternals that has code for unlocking VHD files. (This will be explained later, this version only works for VHD/Images mounted as a disk)
- A good amount of free space.
- Basic knowledge of navigation Command Prompt.
- [ffu2vhdx](https://github.com/gus33000/Ffu2Vhdx)
- [img2ffu](https://github.com/WOA-Project/img2ffu)


## Setting up your environment

- Extract the archive to a folder, ideally on the root of a drive (Below is my example):
e.g: 
F:\aos\os
F:\aos\Tools
F:\aos\OEMInput.xml

- You will have to compile the WPInternals source with one small edit:

Inside "TestCode.cs" you need to modify the line:

	MassStorage MassStorage = new MassStorage("\\\\.\\PhysicalDrive6");
	
Replace the 6 with the number of the drive that the VHD will be when it's mounted later on in the guide.


## Prepping and starting the build

- Now you'll need to edit OEMInput.xml to adjust the path for each "AdditionalFM" listed.
	(You will see AndromedaAppsFM.xml is commented out, this is because the appx files for that Feature Manifest are not present.)

- Once you have made those changes, Open Command Prompt as Administrator and navigate to the ADK tools, these are usually found in if installed with default settings:
  "C:\Program Files (x86)\Windows Kits\10\tools\bin\i386\"

- Now you are able to run the following command to start the build process:

For FFU:

	imggen.cmd F:\aos\flash.ffu F:\aos\OEMInput.xml "F:\aos\os\bin\armfre\prebuilt" arm

For VHDX
You need to remove "<Feature>GATEASSIST_STATESEPARATED</Feature>" from the OEMInput.xml ontherwise it'll fail finalising the image.

	imggen.cmd F:\aos\flash.vhdx F:\aos\OEMInput.xml "F:\aos\os\bin\armfre\prebuilt" arm

It should go through and build the image, If you get errors then either look at the red error text, or try a different ADK tools.




## Making the FFU bootable/usable:
Now this part is the more of a pain but it has to be done.


- *(Skip this if you made a vhdx instead of FFU)* Navigate with the Command Prompt to where you saved ffu2vhdx, and convert the new ffu to vhdx.

- Mount the new vhdx

- Mount HACK_EFIESP and VIRT_EFIESP with either the third party disk management tool of your choice, or Powershell (I use software but it's your choice).

- Double check the BCD inside HACK_EFIESP and make sure "testsigning on" and "nointegritychecks on" are set for the {default} boot option.

- (Optional) In %HACK_EFIESP%\Windows\System32\Boot you can rename and replace ffuloader.efi with developermenu.efi to let you access mass storage when holding Volume Up when powering on.

- Copy everything from `HACK_EFIESP` to `VIRT_EFIESP` using an elevated Command Prompt *(Replace %HACK_EFIESP% and %VIRT_EFIESP% with their drive letters)*:
```
	xcopy /cheriky %HACK_EFIESP%\* %VIRT_EFIESP%\
```
- Now we need to replace the following files with the versions from your phone, these can be found in %MainOS%\Windows\System32:
	- qcadsp8992.mbn
	- qccpe8992.mbn
	- qcdsp1v28992.mbn
	- qcdsp28992.mbn
	- qcvss8994.mbn
	
*To do this you need to either take full control of the files from Right Click > Properties > Security, Or open Command Prompt as TrustedInstaller to replace with the command line.*


- Now we need to use the modified WPInternals to unlock the mounted vhd file with Command Prompt as Administrator:
	F:\Tools\WPinternals-f438d0dd71d86c1e24e2acd8756e7c76a18c443b\bin\Debug\WPinternals.exe -Test
	
	If it goes to plan, it will confirm unlock was successful.


## Rebuilding the FFU:

When the previous section is completed, we can navigate to where you extracted img2ffu and run the below command:
```
  img2ffu.exe --ffu-file "f:\aos\Talkman-18948.ffu" --block-size 32768 --sector-size 512 --plat-id "Microsoft.MSM8992.P6218" --os-version 10.0.18948.1001 --full-flash-update-version V1 --img-file "F:\ffu2vhdx\Store0_VenHw(B615F1F5-5088-43CD-809C-A16E52487D00).vhdx" --device-path VenHw(B615F1F5-5088-43CD-809C-A16E52487D00) --is-fixed-disk-length false --blanksectorbuffer-size 100 --excluded-file .\provisioning-partitions.txt
```
Now you should finally have a bootable FFU
	
- After flashing, let first boot fail, as the device reboots, hold Vol Down until you see the "!", then reset the phone. After a few more reboots it should eventually boot.
	


**WARNINGS: The binary packages inside this pack are taken from the 17110 FFU, I can't guarentee they will work with every 950, I have only confirmed RM-1104 to work so far (DBI, SBL1, TZ, TZAPPS, HYP etc), I will not be held responsible if you don't know what you're doing and brick your device**

  
	
## Troubleshooting
	
- If you boot straight to FFULoader, check VIRT_EFIESP's BCD. if it still happenes, replace it with the one from Tools folder in this pack. (I havent tested if both EFIESP partitions need to match, but the OS boots from VIRT_EFIESP)
- Camera is not yet working.
- RIL/Cellular may not work.
- Re-flashing this build after it has booted and currently installed may not work (Endless bootloop), Reflash a different FFU for your device before reflashing 18948.


## Credits
- BlueRain for dumping and releasing this build
- Tourniquet88 for working with me to fix up and test this build
- WOA-Project's [wskcbsgen](https://github.com/WOA-Project/wskcbsgen) tool
- pinguin2001 for brainstorming and other stuff
