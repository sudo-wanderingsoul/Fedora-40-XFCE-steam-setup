# Fedora-42-XFCE-steam-setup
Personal notes about how to set up Nvidia drivers and steam on my laptop
## Pre-install notes
* This is my personal experience on my specific laptop (Dell G5 5590) so your mileage may vary
* I would also encourage you to look at https://github.com/devangshekhawat/Fedora-40-Post-Install-Guide as they have some good info and commands there
* This guide (notes really) uses Fedora 42 workstation XCFE spin, you might have an easier time using the default gnome distro
## Fedora reimage notes
* I used a USB-C thumbdrive I picked up from micro center to install Fedora (used my thunderbolt port, after I set it to allow bootable media in the BIOS)
* Using a USB-C (instead of USB-A) saved me because I was able to re-image my thumbdrive from my phone using the 'Flasher' app when I borked up my laptop.
### Flasher app subsection (optional if you can't use your main device to re-image the USB)
* Download the 'Flasher' app from the google play store (Android specific)
* Download the Fedora image you want to use
* Plug in the USB-C directly into your phone (assuming it has one USB-C port)
* **_UNMOUNT THE USB STICK FROM YOUR DEVICE_**
* Launch the 'Flasher' app
* Select the image you want, select the USB drive, and image the drive
* Done (or at least what worked for me)
## Actual install notes
Assumes you have already installed Fedora 40 workstations XFCE spin, sources I used below
- [RPMFusion how-to source](https://rpmfusion.org/Howto/NVIDIA "RPM Fusion Instructions")
- [Fedora Quick Docs source](https://docs.fedoraproject.org/en-US/quick-docs/set-nvidia-as-primary-gpu-on-optimus-based-laptops/ "Fedora Docs")
### _Install RPM fusion free and nonfree repos_
```bash
sudo dnf install https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
sudo dnf install https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```
### _Update all existing Fedora packages then reboot to use latest kernel_
```bash
sudo dnf -y update
sudo dnf -y upgrade --refresh
reboot
```
### _Install nvidia drivers after reboot_
```bash
sudo dnf install gcc kernel-headers kernel-devel akmod-nvidia
```
[Link to specific Fedora how-to page](https://docs.fedoraproject.org/en-US/quick-docs/set-nvidia-as-primary-gpu-on-optimus-based-laptops/#_step_5_wait_for_the_kernel_modules_to_load_up)
* **_IMPORTANT: Confirm the akmod process has completed building the driver, this takes around 5-10 minutes_**
* Can use `btop`, `htop`, `top`, or `ps` to view running processes
* **_ONCE AND ONLY ONCE THE NVIDIA DRIVER HAS BEEN BUILT, RUN THE FOLLOWING COMMANDS_**
```bash
sudo akmods --force
sudo dracut --force
reboot
```
## _The following are optional packages_
### **Install NVIDIA Cuda support (optional but recommended)**
* NOTE: nvidia-smi command from package is very useful
```bash
sudo dnf install xorg-x11-drv-nvidia-cuda
```
### **Install additional support/diag packages (optional)**
* Packages to help with diagnostic data just in case something goes wrong
```bash
sudo dnf install xrandr inxi
```
### _Switcheroo control package (optional)_
* Package in case you want to be able to spectify which GPU a program launches on
```bash
sudo dnf install switcheroo-control
```
### _Switch to nvidia as primary GPU (optional)_
* The following is a third-party repo/package, proceed with caution in that knowledge
- Found https://github.com/bayasdev/envycontrol was recommended on a reddit thread and it worked like magic on Fedora
- Modes are - "integrated", "hybrid", "nvidia"
```bash
sudo dnf copr enable sunwire/envycontrol
sudo dnf install python3-envycontrol
sudo envycontrol -s <MODE>
reboot
```
## _Steam install section_
### _Install steam_
```bash
sudo dnf install steam
```
### _Update steam shortcut / switcheroo-control use example (optional)_
#### Note: Use this only if you have to, I have since been able to set nvidia as my primary display, see above
I needed to replace the command that steam.desktop ran to ensure it used my second (Nvidia) GPU\
Location of steam.desktop: /usr/share/applications/steam.desktop\
You can use an editor of your choice but I used `vim`
```bash
vim /usr/share/applications/steam.desktop
```
* Line to replace (line 30): `Exec=/usr/bin/steam %U`
* Replacement with: `Exec=sh -c "/usr/bin/switcherooctl -g 1 /usr/bin/steam %U"`
* Save and exit steam.desktop
* You should be able to launch steam via the shortcut and it use the correct GPU\
If anything doesn't work after this, use the below diagnostic section to help
## _Diagnostic commands and general help section_
##### Check which processes are using the Nvidia GPU
```bash
nvidia-smi
```
##### Get eay-to-read system info (for diag or supprt via forums)
```bash
inxi -Fzxx
```
##### List what GPUs switcheroo sees
```bash
switcherooctl list
```
##### Google search and fedora forums helped me add missing bits of info
* If all else fails, google search specific symptons aka "fedora nvidia not showing GPU" or search the [Fedora forums](https://discussion.fedoraproject.org)
###### Forums to start debuging with
* https://discussion.fedoraproject.org/t/optimus-setting-the-nvidia-gpu-as-primary-rpmfusion-in-fedora-32-workstation/72796
* https://www.reddit.com/r/Fedora/comments/ga1ek6/optimus_setting_the_nvidia_gpu_as_primary/
