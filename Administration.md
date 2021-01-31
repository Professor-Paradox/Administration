- [Operating System](#operating-system)
  - [Boot Process](#boot-process)
    - [Bios/Cmos/Uefi](#bioscmosuefi)
    - [MBR/GPT](#mbrgpt)
    - [GRUB](#grub)
    - [Kernel](#kernel)
    - [Init](#init)
- [Windows Commands](#windows-commands)
- [Linux Commands](#linux-commands)


# Operating System
Complete guide of operating System.
## Boot Process
### Bios/Cmos/Uefi
**Basic Input Output System:**  
This is the primary software/hardware layer that is triggered after the power button is pressed.   
Bios is written to a read only(non-volatile) chip,often referred to as pc firmware.    
Performs (Power on Self test(post)) to check integrity(is it working) of all components present.    
Searches for a boot device   
loads the boot program in to memory and gives control to it.  
Provides abstraction for i/o hardware,so o.s/kernel will talk to the bios to perform operations.    
Modern O.S's don't use this abstraction anymore,and access the devices directly.   

**Complementary metal oxide semiconductor:**    
A volatile chip that stores all the custom settings of bios in it.   
to retain all the settings of bios in case of power loss, a cmos-battery is used.
if the battery is removed all the settings in the cmos chip will be erased(restores default settings)    

**Unified Extensible Firmware Interface:**   
New revision of bios.   
can use mouse(better U.I)   
supports secure boot: stops unsigned drivers to load in to kernel,(stops root kits)    
Bios/legacy compatibility   
remote diagnostics



### MBR/GPT
**Partitioning:**   
The process of dividing the hard Disk/SSD in to multiple parts(logical Drives).  
this is used to create an illusion of having more drives(logical) to the O.S.   
Was Majorly used due to filesystem limitations of the time(1995).    
Now a days mostly used to store all the personal data in one partition and system data in other. 

**File System:**    
To organize and track all the files/data in a drive a *file system* is used.     
Using clusters we create a structure to operate over files/data.     
Different File System's use clusters in different ways,and have features for various purposes.        
Now a days all file system's are hierarchical file systems,(folder inside a folder with unlimited depth)         

* **FAT:**
   File Allocation Table,FAT is a simple File System,gives clusters ID's and stores files in it.    
   2GB partition limit.     
   results in Slack(wastage of space in a cluster,storing 10 Byte file in 256 Byte cluster)    
* **FAT 16/32**     
   32bit implementation of FAT,supports file size of 4GB.Used for memory cards,pendrives.     
* **exFAT**     
   Extended version of FAT for better support,these day's used for massive portable drives.Wide support on multiple devices.     
* **NTFS**     
  New Technology File System:used by windows system drives,and have      
  * security,   
  * journaling,   
  * efficiency,    
  * encryption,   
  file permissions features.      
* **HFS**     
   Apples own File system, with journaling,security,encryption,efficiency features. but not supported out side of apple eco-system.     
* **ext4**     
   extended file system has journaling,encryption,file permissions.default in linux.     
* **ZFS**     
   Zed File System,better data protection by letting volume manager control hard disks,available in linux     
* ****
**Default Values**:(remember gpt drive has a limit of 9.4 ZetaByte limit)        

| File System | cluster size(4K Sector) | Partition(Theory/RealWorld) | File Size(MAX)    |
| ----------- | ----------------------- | --------------------------- | ----------------- |
| NTFS        | 4KB  (default for all)  | 18 ExaBytes(256TB)          | 16 ExaBytes(16TB) |
| HFS+        | 4KB                     | 8EB(8EB)                    | 8EB (8EB)         |
| FAT32       | 4KB                     | 2TB(32GB)                   | 4GB(4GB)          |
| exFAT       | 4KB                     | 128PB                       | 128PB             |
| ext4        | 64KB                    | 1 ExaByte(1EB)              | 16TB(16TB)        |


1.  **sector**/logical block      
      A sector is a smallest physical storage unit of a disk, it is almost always 512 Bytes in size.     
      sector size is defined while the drive is beign manufactured.      
      Newer Drives are using sectors of size 4096 Bytes(4k Sectors),called Advaned Format.      
2.  **cluster**     
      A cluster is a group of continuous sectors, always a power of 2.     
      This represents the smallest disk space needed for a file.      
      A file of size 1500 Bytes will require 2048 Bytes in 4 or 2<sup>2</sup> sectors,a file of size 3000 Bytes require 8 sectors(8 * 512 = 4096 Bytes or 2<sup>3</sup> sectors)    
      A cluster with 1 or more sectors, reduce fragmentation(since a file can be written to the large space).   
      cluster size will be optimized for each file system.So can be changed on demand/need.   
3.  **Fragmentation**    
      When a file is written non-continuously(editing a 3000 Byte file to 5000 Byte file, the new data will be written to next available cluster),the file would be scattered across the disk.   
      This is called fragmentation,Results in slow read speeds of files(hard drives only, ssd's don't suffer from fragmentation).      


**Master Boot Record:**     
It is located in the 1st sector of the bootable disk. Typically /dev/hda, or /dev/sda
MBR is less than 512 bytes in size.    
It has three components, primary bootloader(a link to actual boot loader),partition table,validation checks.   
So, in simple terms MBR loads and executes the GRUB boot loader.   
Supports only 4 partitions 
Single Drive size/Partition limit is 2 Tera Bytes(MAX LIMIT)(remember a single physical partition is a single logical drive)     
In mbr based systems with bios, the first partition has the bootloader needed,so that will invoke the actual bootloader and loads kernel to memory.

**GUID(Global Unique Identifier) Partition Table**   
Each partition has a Guid so we can have unlimited partitions in a drive(s).
Guid drives can be of any size limited only by the operating system,file system.
Number of patitions in Each Operating system is

| O.S     | Partition |
| ------- | --------- |
| Windows | 128       |
| Mac Os  | 128       |
| Linux   | 128       |
| Android | 40+       |
   
GPT stores *Cyclic Redundant Check*(CRC) for error checking and recovery helping.(mbr has none)     
Has compatibility with MBR(protective MBR shows the entire drive as an single partition)     
Single Drive/Partition size limit is 9.4 ZetaBytes(9.4 ZB = 9,400,000,000 TB,2<sup>64</sup> sectors)
Types of partitions:    
/efi:stores all the kernels of the operating system     
when a gpt system is used with uefi,the efi system partition(ESP) is called and a bootloader is loaded in to memory.     
> In dual boot grub is used. efi partition has linux kernel and microsoft windows kernel,grub then offers options to which kernel to load.     


### GRUB   
**Grand Unified Bootloader:**    
A *Bootloader* is the medium that is initiated by the bios/uefi and selects a kernel from partition(s), and gives control of booting to it. 
Grub displays a splash screen,waits for few seconds for input, if none is given performs default operations.     
Can choose from multiple kernels,operating systems.    
GRUB has the knowledge of the filesystem.     
Grub configuration file is /boot/grub/grub.conf    
So, in simple terms GRUB just loads and executes Kernel and initrd images.    

all files needed for bootloader(grub) are present in the /boot directory and has following files 
* [initrd](#init)
* efi files (bootloader files from other o.s's in multi boot)
* [grub](#grub) for all grub related files
* vmlinux - uncompressed format of linux [kernel](#kernel)
* vmlinuz - compressed format of linux kernel 
* any other config files required for os boot.

### Kernel
Mounts the root file system as specified in the “root=” in grub.conf    
Kernel executes the /sbin/init program.Since init was the 1st program to be executed by Linux Kernel, it has the process id (PID) of 1.    
initrd stands for Initial RAM Disk.
initrd is used by kernel as temporary root file system until kernel is booted and the real root file system is mounted. It also contains necessary drivers compiled inside, which helps it to access the hard drive partitions, and other hardware.

### Init
**Initrd**
Initial ram disk is the temporary file system used to load modules for all the peripherals and drives,after an o.s is loaded this initrd is removed and given control to os.


# Windows Commands

**(just commands no explanation)**
**all are powershell commands unless mentioned**

1. ls(lists directories)
   1. -Filter(filters files with a pattern)
   2. -Recurse(selects multiple files in a directory with recurse)
   3. -Verbose(gives details about the command)
2. Get-Help ls
   1. -Full (gives all available details of a command)  
   2. 
3. pwd(print working directory)
4. cd(change directory)
   1. cd ..(up one level)   
   2. cd ~(home directory shortcut)
5. history(list all previous commands used)
6. #command(search previous used command)
   1.  #mkdir(use up and down arrow to search similar commands in history) 
7. clear(clears output on the screen)
8. cp(copy files from source to destination)
   1. *(pattern to do multiple tasks once) cp *.jpg ~/(example)
   2. -Recurse(selects multiple files in a directory with recurse)
   3. -Verbose(gives details about the command)
9.  mv(move files/name of file from source to destination)
    1. -Recurse(selects multiple files in a directory with recurse) 
10. rm(removes files without recycle bin
    1.  -Recurse(selects multiple files in a directory with recurse)
    2.  -Force(gives additional previlages to remove the file/folder)
11. cat(concatenates text mostly to display text)
12. more(shows all text with respect to screen size)
    1.  -Head 10 (shows first 10 lines)
    2.  -Tail 10 (shows last 10 lines)
13. start(starts a program and gives specified input to it)
14. Get-Alias(shows the true command of a alias)Get-Alias ls==>Get-ChildItem
    1.  
15. /?(get-help alternative for command prompt)
16. Select-String(searches for the pattern provided in a file)
    1.  
17. ">"(redirector operator,makes output flow to a file/other program)
18. ">>"(same as > without overwriting but appending to exisiting file)
19. "|"(give the output of one program to other programs input)
20. 2>(stderror)(used to log errors to a log file)(redirecto the stream)
21. $null(can redirect data to this variable to not store anything)
22. 








# Linux Commands
1. mkdir(makes a new directory)
2. ls(list directories)
   1. file names with spaces should quoted around
3. history(list all previous commands used
   1. ctrl-R(to open history) 
4. clear(clears output on the screen)
5. *(pattern to do multiple tasks once)
6. cp(copy files from source to destination) cp *.jpg ~/(example)
   1. -r(copy multiple files in a directory with recurse)
   2. -v(gives details about the command)
7. mv(move files/name of file from source to destination)
   1. -r(recursively move all files)
   2. -f(forces with elevated previlages)
8. rm(removes files without recycle bin)
9.  cat(concatenates text to screen)
10. less(replacement of more command,shows text)
    1.  /(searching for a word)
11. head(gives first 10 lines of a file)
12. tail(gives last 10 lines of a file)
13. nano(super useful text editor)
14. ">"(stdout)(redirector operator,makes flow to a file/other program)
15. ">>"(same as > without overwriting but appending to exisiting file)
16. "|"(give the output of one program to other programs input)
17. grep(used to search for text in a given data with matching pattern)
18. 2>(stderror)(used to redirect the error of a command to some file)
19. "<"(stdin)(used to take input from a file to a command)
20. /dev/null(a null file with nothing it in and can be used to redirect output/erros to it to not store anything)

seq(prints a sequence of numbers with parameters)used in range related applications.0
echo(to print content on screen)we can used double quotes to enclose the content to print or just give the content itself.
read(to take input from the user)
ls(list files)
cd(change directory)
    cd ..(up a directory)
    cd (home directory)
    cd ~(home directory)

pushd(to go in a directory)
popd(to go back to first directory)for ease and not remembering the direcotry place
file(to check the type of file)
locate(search for a file)(need to update database)
    sudo updatedb
which(searching the location of a package/program)
history(shows the last 1000lines of commands run in the terminal)
whatis(used to get the introduction of a package/program)
apropas(to search for a package with relevant to given input)
man(to read the manual pages of package/command, shows how to use)
cp(copy first file into second file)
mv(used to move files and rename)takes first file and keeps it in the second place
rm(powerful, removes files and directories)
glob(* in the command that fills and searches is glob,? searches only one character)
rmdir(used to remove empty directories)
cat(used to concatenate read and add text to files)(small files)
    > filename(will create a new file/overwrite extending exisiting one)
    >> filename(will add to the end of file)
more(text reader that shows document in page order with space button)
less(advanced than more command can use arrow,space,search for)
nano(text editor in terminal)
|(piping taking output from one program, giving it to the input of another program)
chmod(changing permission for user,group,everyone to give read(4) write(2) execute(1) access.)chmod 755 or +x
killall(kills the process name given)
which    
touch     
find 

<!-- 
# SSH Cheat Sheet
## This sheet goes along with this [SSH YouTube tutorial](https://www.youtube.com/watch?v=hQWRp-FdTpc&t=1270s)

### Login via SSH with password (LOCAL SERVER)
```$ ssh brad@192.168.1.29```

### Create folder, file, install Apache (Just messing around)
```$ mkdir test```

```$ cd test```

```$ touch hello.txt```

```$ sudo apt-get install apache2```


### Generate Keys (Local Machine)
```$ ssh-keygen```

### Add Key to server in one command
```> cat ~/.ssh/id_rsa.pub | ssh brad@192.168.1.29 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >>  ~/.ssh/authorized_keys```

### Create & copy a file to the server using SCP
```$ touch test.txt```
```$ scp ~/test.txt brad@192.168.1.29:~```


## DIGITAL OCEAN

> Create account->create droplet

### Create Keys For Droplet (id_rsa_do)
```$ ssh-keygen -t rsa```

> Add Key When Creating Droplet

### Try logging in
```$ ssh root@doserver```

### If it doesn't work
```$ ssh-add ~/.ssh/id_rsa_do```
(or whatever name you used)

### Login should now work
```$ ssh root@doserver```

### Update packages
```$ sudo apt update```

```$ sudo apt upgrade```

### Create new user with sudo
```$ adduser brad```

```$ id brad```

```$ usermod -aG sudo brad```

```$ id brad```

### Login as brad
```> ssh brad@doserver```

### We need to add the key to brads .ssh on the server, log back in as root
```$ ssh root@doserver```

```$ cd /home/brad```

```$ mkdir .ssh```

```$ cd .ssh```

```$ touch authorized_keys```

```> sudo nano authorized_keys```
(paste in the id_rsa_do.pub key, exit and log in as brad)

### Disable root password login
```$ sudo nano /etc/ssh/sshd_config```

### 4.2.10. Set the following
```PermitRootLogin no```

```PasswordAuthentication no```

### 4.2.11. Reload sshd service
```$ sudo systemctl reload sshd```

### 4.2.12. Change owner of /home/brad/* to brad
```$ sudo chown -R brad:brad /home/brad```

### 4.2.13. May need to set permission
```$ chmod 700 /home/brad/.ssh```

### 4.2.14. Install Apache and visit ip
``` $ sudo apt install apache2 -y```

## Github

### Generate Github Key(On Server)
``` $ ssh-keygen -t rsa```
(id_rsa_github or whatever you want)

## Add new key
```$ ssh-add /home/brad/.ssh/id_rsa_github```

## If you get a message about auth agent, run this and try again
```$ eval `ssh-agent -s````

## Clone repo
```$ git clone git@github.com:bradtraversy/react_otka_auth.git```

## Install Node
```$ curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -```

```$ sudo apt-get install -y nodejs```

## Install Dependencies
```  $ npm install ```

## Start Dev Server and visit ip:3000
```$ npm start```

## 4.10. Build Out React App
``` $ npm run build```

## 4.11. Move static build to web server root
``` $ sudo mv -v /home/brad/react_otka_auth/build/* /var/www/html```--->




