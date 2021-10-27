# Arch linux install
So basically, this is the best most simplified yet still complete documentation ever written. you would have to be a gill-less fish to strugle with this installation after following this guide. If you are confused and don't know what is going on, just keep reading. When in doubt: **do exactly as I say**. 
## Installing the iso
just get it somehow

### **Booting** 
>I am using VMware workstation as my hypervisor, I would suggest you do the same.

Create a new virtual machine and use Debian10.x64

Because VMware does not allow you to boot into UEFI mode you will need to make changes to the vmx file

> After you finish the setup of the virtual machine, it would have created a folder for your Machine in the location that you specified. There, you should find a .vmx file <i>**if your host machine is a Windows OS, you can click view and check the box that says show file extentions. This will help you locate which file is the vmx file**,</i>

On the second line of the vmx file write: 

`firmware = "efi"`

Save the file and power on your virtual machine. Once you are inside of the virtual machine it should ask how you want to boot. You should select the first option of UEFI 



# Inside the VM 
### **Connecting to the Internet**

To check your network cards type the command:
`# ip linK` You should have at least two cards. 

>If you are using VMware, and your host machine is connected to the internet, your virtual machine should automatically be able to connect to the internet.

Now to make sure that we can reach the internet you should ping google this should confirm your internet connectivity
 <br> `ping google.com` <br>  
 > *You should see the terminal start to send packets to google to test connectivity*
  
   use Ctrl-C to stop pinging. 

   ### **Update system Clock**
use timedatectl to update the system time 
~~~
# timedatectl set npt-true
~~~
then you want to check the status of your time with<br> 
~~~
 # timedatectl status
~~~

If your time zone is not accurate, use `# timedatectl list-timezones` to list all available timezones and then use <br> `# timedatectl set-timezone < your timezone >` 

### **Partition the disks**

Use `# fdisk -l` to list all current disks/partitions. It is okay if there are partitions that start with `rom`,`loop`,or `airoot`

Use `# cfdisk /dev/sda` to navigate to the inside of your disk drive

To create a the first partition (the boot or efi partition) "sda1" press **Enter**  for a new partition then type `500M`. After, you should select the type menu and change the partition type to `EFI 16/32/64`.

Next you will create the root partition "sda2" which houses your file system. However, this partition will have `18.5G` for storage, and you will **not** change the partition type.

Lastly, create the third and final partition the "sda3" (or swap) partition and give it `1G` of storage and make it type `Linux Swap`

Next, create a second partition that uses the remainder of the space on the disk.

**Configure the swap file system**

Create the swap system
~~~
# mkswap /dev/sda3
# swapon /dev/sda3
~~~

**Configure the root file system**

Format the root file system as ext4
~~~
# mkfs.ext4 /dev/sda2
~~~

**Configure the EFI file system**

Format the EFI file system as a FAT32
~~~
mkfs.fat -F32 /dev/sda1
~~~

### **Mount the Partitions**
 Mount the root partition

~~~
# mount /dev/sda2 /mnt
~~~
Create the directory for the EFI partition and mount it 
~~~
# mkdir /mnt/boot 
# mount /dev/sda1 /mnt/boot
~~~

create a new directory `/mnt/boot` with mkdir and mount sda1 there

# Installation 

### **Install packages**

Install the essential packages with 

~~~
# pacstrap /mnt base linux linux-firmware
~~~

#### **Configure the system**

~~~
# genfstab -U /mnt >> /mnt/etc/fstab
~~~

#### **Change root into new system**

~~~
# arch-chroot /mnt
~~~

install a text editior while inside new system.

~~~
# pacman -S nano
~~~

### **Basic Configurations**

#### **Timezone**
Change the timezone 

~~~
# ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
~~~

#### **HwClock**
set up hclock 

~~~
# hwclock --systohc
~~~ 
#### **localization**
Use nano to modify `/etc/locale.gen` and uncomment en_US.UTF-8 UTF-8

Then create and modify a new file `/etc/locale.conf` to read:
~~~
LANG=en_US.UTF-8
~~~
**Network Configuration**
create and modify the hostname file 
~~~
nano /etc/hostname
-------------------
myhostname
~~~

**Set the root password**
~~~
# passwd
~~~
**Install "GRUB" Bootloader**

Use pacman to install the grub package and then install the boot loader onto the boot disk
~~~
# pacman -S grub efibootmgr
# grub-install --target=x86_64-efi --efidirectory=/boot --bootloader-id=GRUB
# grub-mkconfig -o /boot/grub/grub.cfg
~~~
>Now your system should be functional. Next, we need to install a Desktop Enviroment to make the system viable for everyday useage.

# Optional: QOL Changes

**Add an additional Shell**

Add zsh using pacman
~~~
# pacman -S zsh
~~~
**Add Color coding to terminal**

Add color to nano
~~~
nano /etc/nanorc 
~~~ 
add line `include "/usr/share/nano/*.nanorc"`

**Install SSH**

Install ssh for the ability to shell into a new gateway 
~~~
# pacman -S openssh
~~~

**Install 
# Adding a Desktop Enviroment
First we must install sudo and vim
~~~
# pacman -S sudo
# pacman -S vim
~~~

Create a new user and set up a password
~~~
# useradd -m -g users -s /bin/bash <username>
----------------------
# passwd
~~~
> Next you need to add the new user to the sudoers list by modifying the sudoers file. 
 ~~~~
 visudo 
 ~~~~
 >This will enable you to edit the sudoers file. Scroll down to where you see the user "root" and under it, Add the name of the user and give it the same permissions then save the file. Use **i** to write to the file and **:wq** to escape
~~~~
username ALL=(ALL) ALL
~~~~
**Install Graphics Server**

 Use pacman to install xorg graphics server
 ~~~
 # pacman -S xorg-server xorg-apps xorg-xinit
 ~~~
 **Install Graphics Driver**
 
 Use pacman to install generic graphics driver 
 ~~~
 # pacman -S xf86-video-vesa
 ~~~
 **Install a display manager** 
 
 use pacman to install sddm for KDE
 ~~~
 pacman -S sddm
 ~~~
 **Install actual Desktop Enviroment**
 
 Use pacman to install the KDE desktop enviroment and the additional tools
 ~~~
 # pacman -S plasma kde-applications
 ~~~
 >This may take a long time depending on your internet connection

 **Configure Graphical boot**
Enable graphics driver to start on boot
~~~ 
# systemctl enable sddm
~~~
**Configure Network Manager**

Use pacman to install the networkmanager then use `systemsctl` to enable the program
~~~
# pacman -S networkmanager
# systemctl enable NetworkManager
~~~


After this you are pretty much done. Now all you need to do is `exit` the chroot and `reboot`. if everything is done properly, your vm should boot into the GUI of KDE plasma 5. You should be able to log into the desktop with your username and password that you created


thx for reading, 

-Myles <3


