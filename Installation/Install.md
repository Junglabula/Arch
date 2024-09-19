# Arch Install
[github step by step](https://github.com/dreamsofautonomy/arch-from-scratch)


# config internet && SSH
Ouvre SSH:

	 sudo systemctl start sshd

## Configure le password root

	ConfigPassword a root: passwd



SSH en root par CMD: ssh root@ipAddress

# Overwrite data on disk & partition

## Flush & rewrite disk

List des disks:

	 lsblk

rewrite le disk name (ex sda):

	 dd if=/dev/urandom of=/dev/sda bs=4096 status=progress

**ps: le rewrite prend longtemps a completer (4-6h pour 1tb), mais le securise**



## Config des partitions
GOAL
| Number | Start (sector) | End (sector) | Size |  Code | Name |
|:-|:-|:-|:-|:-|:-|
|   1|           2048 |        1050623 |  512.0 MiB |  EF00|  EFI system partition |
|   2 |        1050624 |        9439231 |  4.0 GiB |    EF02|  BIOS boot partition |
|   3 |       9439232 |       42993663 |  16.0 GiB |   8200|  Linux swap |
|   4 |      42993664 |      177211391 |  64.0 GiB |   8300|  Linux filesystem |
|   5 |       177211392 |      268433407 |  43.5 GiB |   8302|  Linux /home |


	gdisk /dev/sda

Pour EFI

	n
	enter
	enter
	+512M
	L                    #Pour afficher la liste
	efi
	ef00
	Afficher se qui a ete fait: P
    p 
Pour BOOT

	n
	enter
	enter
	+4G
	L
	boot
	Trouver ef02 BIOS boot partition
	ef02
	Afficher se qui a ete fait: p 
Pour swap

	n
	enter
	enter
	+16G
	L
	swap
	choisir 8200 Linux swap		
	8200
Pour Filesystem //laisser de l'espace pour /home si vous voulez qu'ils soient dans une autre partition

	n
	enter
	enter
	choisir
	L
	default
	8300
Pour /home

	n
	enter
	enter
	enter
	L
	home
	8302
Pour sauvegarder le tout:
	
	w
	y


# setup filesystem
EFI

	mkfs.fat -F32 /dev/sda1 
boot

	mkfs.ext4 /dev/sda2

## btrfs
root

	mkfs.btrfs -L root /dev/sda4

home

	mkfs.btrfs -L home /dev/sdb1

swap

	mkswap /dev/sda3

# mount
enable & swap

	swapon /dev/sda3
	swapon -a
Mount root

	mount /dev/sda4 /mnt
Create home et boot

	mkdir -p /mnt/{home,boot}
Mount boot

	mount /dev/sda2 /mnt/boot
Mount home

	mount /dev/sdb1 /mnt/home
Create efi folder

	mkdir /mnt/boot/efi
Mount efi

	mount /dev/sda1 /mnt/boot/efi

### Ce que lsblk devrait afficher

	root@archiso ~ # lsblk
	NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
	loop0    7:0    0 790.3M  1 loop /run/archiso/airootfs
	sda      8:0    0   128G  0 disk
	├─sda1   8:1    0   512M  0 part /mnt/boot/efi
	├─sda2   8:2    0     4G  0 part /mnt/boot
	├─sda3   8:3    0    16G  0 part [SWAP]
	└─sda4   8:4    0 107.5G  0 part /mnt
	sdb      8:16   0    96G  0 disk
	└─sdb1   8:17   0    96G  0 part /mnt/home
	sr0     11:0    1   1.1G  0 rom  /run/archiso/bootmnt


# install arch

	pacstrap -K /mnt base linux linux-firmware

## save filesystem

	genfstab -U -p /mnt > /mnt/etc/fstab


# Terminer l'installation 

## se connecter en root sur l'installation fait precedement

	arch-chroot /mnt /bin/bash
## Install package

	pacman -S base-devel

## Install text editor
neovim or nano

	pacman -S neovim
	pacman -S nano

## Install Grub & efimanager on boot partition
### Download

	pacman -S grub efibootmgr	
### Install

	grub-install --efi-directory=/boot/efi


### configure

	nano /etc/default/grub
Modifier la ligne inscrit: GRUB_CMDLINE_LINUX_DEFAULT"... pour quel ressemble a ceci:
	
	GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet root=/dev/sda4"

	ctrl+x y enter
Aller chercher le uuid de sda4

	blkid /dev/sda4
Copier le uuid

retourner dans la config et modifier le chemin du disk pour le uuid

	nano /etc/default/grub
Meme ligne que tout à l'heure

	GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet root=UUID=921410f8-6a98-40d7-892c-3b84b18d40b1"


### Grub/EFI - push la mkconfig vers la config

	grub-mkconfig -o /boot/grub/grub.cfg
	grub-mkconfig -o /boot/efi/EFI/arch/grub.cfg


## Time/NTP
### Timezone

	ln -sf /usr/share/zoneinfo/Canada/Eastern /etc/localtime
### NTP
En dessous de [Time] ajouter ou modifier pour que c'est 2 lignes soient ajoutés

	nano /etc/systemd/timesyncd.conf

	[Time]
	NTP=0.arch.pool.ntp.org 1.arch.pool.ntp.org 2.arch.pool.ntp.org 3.arch.pool.ntp.org
	FallbackNTP=0.pool.ntp.org 1.pool.ntp.org	

### service
Enable le time service

	systemctl enable systemd-timesyncd.service
### locale

	nano /etc/locale.gen
Choisir une langue, trouver la et enlever le commentaire

	#en_BW.UTF-8 UTF-8
	#en_BW ISO-8859-1
	en_CA.UTF-8 UTF-8
	#en_CA ISO-8859-1
	#en_DK.UTF-8 UTF-8

generate language

	locale-gen

### Language entry

	nano /etc/locale.conf

	LANG=en_CA.UTF-8

### hostname

	nano /etc/hostname

ecrire un now et sauvegarder

	patate

### setup root password

	passwd


### Install zsh

	pacman -S zsh

### Create user

	useradd -m -G wheel -s /bin/zsh mathi
setup password for user
	
	passwd mathi

### setup permission

	export EDITOR=nano
	visudo
Uncomment at the end, the line: 

	%wheel ALL=(ALL:ALL) ALL

### network
download

	pacman -S networkmanager
Enable on boot

	sudo systemctl enable NetworkManager

## Install GUI





## install microcode

	pacman -S intel-ucode
save grub file

	grub-mkconfig -o /boot/grub/grub.cfg



### quitter et reboot
exit root

	exit

unmount l'install

	umount -R /mnt
reboot

	reboot now

















