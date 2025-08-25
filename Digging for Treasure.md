---
tags:
  - ctfs/challenges
ctf: "[[BrunnerCTF 2025]]"
areas:
  - "[[Forensics]]"
solved:
---

> [!info] Prompt
> **Difficulty:** Beginner  
**Author:** Emil8250
> 
> Last night I was out in the darkest of forests, located on Funen, digging for treasure. Lo and behold, I found this mysterious envelope with a note attached: _"If you've found this by accident, please leave it where you found it. This is the best brunner recipe to ever be made, and the local baking-cartels have been looking for it, you'll be in great danger for possessing it."_
> 
> The envelope contained a USB-drive. Naturally, I needed to do an [autopsy](https://www.autopsy.com/download/) on this incredible find and plugged it straight into my PC, but it seems like all the good stuff's missing?
> 
> File download included: `forensics_digging-for-treasure.zip`.

# Walk Through

The prompt wants us to install Autopsy, which is apparently a Java-based GUI over [The Sleuth Kit (TSK)](https://sleuthkit.org/).

I haven't used either of these tools before. However, I'd prefer not to install the Autopsy GUI on my system if I can avoid it. I don't have a VM ready to go and I'd rather not bother with it at the moment, but that's an alternative solution. I'm playing this CTF on my main workstation at the moment.

Anyway, I'll just install TSK from the standard apt repositories and yank the thing.

I am familiar with the Linux command line. Looks like the GUI doesn't do anything the CLI TSK tool can't do. I'll use ChatGPT to help me learn this tool quickly and see if I can find the flag.

## Poking around

### Initial look at files from given archive

```
❯ ls forensics_digging-for-treasure
recipe_usb.E01  recipe_usb.E01.txt

❯ file forensics_digging-for-treasure/*
forensics_digging-for-treasure/recipe_usb.E01:     EWF/Expert Witness/EnCase image file format
forensics_digging-for-treasure/recipe_usb.E01.txt: Unicode text, UTF-8 (with BOM) text, with CRLF line terminators
```

#### Text file - `recipe_usb.E01.txt`:

```
Created By AccessData® FTK® Imager 4.7.1.2 

Case Information: 
Acquired using: ADI4.7.1.2
Case Number: BrunnerCTF
Evidence Number: Digging For Treasure
Unique Description: 
Examiner: You!
Notes:  

--------------------------------------------------------------

Information for C:\Users\User\recipe_usb:

Physical Evidentiary Item (Source) Information:
[Device Info]
 Source Type: Physical
[Drive Geometry]
 Cylinders: 489
 Tracks per Cylinder: 255
 Sectors per Track: 63
 Bytes per Sector: 512
 Sector Count: 7.856.127
[Physical Drive Information]
 Drive Model: SanDisk Cruzer Blade USB Device
 Drive Serial Number: 0875920F047172B7
 Drive Interface Type: USB
 Removable drive: True
 Source data size: 3835 MB
 Sector count:    7856127
[Computed Hashes]
 MD5 checksum:    3e6daf20ce837abc27c6841d379eae66
 SHA1 checksum:   20f3565203bccc47cbf6045aead1ac7fc2a10d39

Image Information:
 Acquisition started:   Wed Aug 13 00:10:23 2025
 Acquisition finished:  Wed Aug 13 00:19:15 2025
 Segment list:
  C:\Users\User\recipe_usb.E01

Image Verification Results:
 Verification started:  Wed Aug 13 00:19:15 2025
 Verification finished: Wed Aug 13 00:19:27 2025
 MD5 checksum:    3e6daf20ce837abc27c6841d379eae66 : verified
 SHA1 checksum:   20f3565203bccc47cbf6045aead1ac7fc2a10d39 : verified

```

It's a text file containing information about the drive that was imaged using the program specified at the top of the file.

#### EWF (EnCase/Expert Witness) file - `recipe_usb.E01`


> [!quote] ChatGPT
> EWF (EnCase/Expert Witness) files are **forensic disk image formats**, commonly used in investigations. They're not a raw byte-for-byte image, so you need special tools to work with them.

## The Sleuth Kit (TSK)


> [!quote] ChatGPT
> TSK itself doesn’t natively understand `.E01` (EWF) containers. Instead, you use **libewf / ewf-tools** to expose the EWF as a **raw disk image**, and then point TSK at that raw image.

### 1. Install libraries and utilities

```
❯ sudo apt install libewf-dev ewf-tools

<snip>
```

### 2. Inspect the image

```
❯ ewfinfo recipe_usb.E01
ewfinfo 20140813

Acquiry information
        Case number:            BrunnerCTF
        Description:            untitled
        Examiner name:          You!
        Evidence number:        Digging For Treasure
        Notes:                   
        Acquisition date:       Tue Aug 12 22:10:23 2025
        System date:            Tue Aug 12 22:10:23 2025
        Operating system used:  Win 201x
        Software version used:  ADI4.7.1.2
        Password:               N/A

EWF information
        File format:            FTK Imager
        Sectors per chunk:      64
        Compression method:     deflate
        Compression level:      no compression

Media information
        Media type:             fixed disk
        Is physical:            yes
        Bytes per sector:       512
        Number of sectors:      7856127
        Media size:             3.7 GiB (4022337024 bytes)

Digest hash information
        MD5:                    3e6daf20ce837abc27c6841d379eae66
        SHA1:                   20f3565203bccc47cbf6045aead1ac7fc2a10d39
```

Apparently, the OS used in this image is Win 201x, and there's no password required.

### 3. Mount the image

```
❯ mkdir recipe_mnt
❯ ewfmount recipe_usb.E01 recipe_mnt
ewfmount 20140813
```

*Note: ChatGPT suggested I create the mount directory in* `/mnt/`*, but we don't need to do that. You can create a mount point in the local directory instead and mount the image to that, as I did.*

Check out the image we just mounted:

```
❯ ls recipe_mnt
ewf1
❯ file recipe_mnt/ewf1
ewf1: DOS/MBR boot sector; partition 1 : ID=0x7, start-CHS (0x0,0,39), end-CHS (0x1e7,254,63), startsector 38, 7839682 sectors, extended partition table (last)
```

### 4. Run TSK tools on the mounted image

`fls` is a tool to list file and directory names in a disk image.

```
❯ fls -r recipe_mnt/ewf1
Cannot determine file system type
```

Perhaps we can determine the file system type ourselves and tell `fls` what it is.

```
❯ fls -f list
Supported file system types:
        ntfs (NTFS)
        fat (FAT (Auto Detection))
        ext (ExtX (Auto Detection))
        iso9660 (ISO9660 CD)
        hfs (HFS+ (Auto Detection))
        yaffs2 (YAFFS2)
        apfs (APFS)
        ufs (UFS (Auto Detection))
        raw (Raw Data)
        swap (Swap Space)
        fat12 (FAT12)
        fat16 (FAT16)
        fat32 (FAT32)
        exfat (exFAT)
        ext2 (Ext2)
        ext3 (Ext3)
        ext4 (Ext4)
        ufs1 (UFS1)
        ufs2 (UFS2)
        hfsp (HFS+)
        hfsl (HFS (Legacy))
```

No dice...

```
❯ fls -f ntfs -r recipe_mnt/ewf1
Invalid magic value (Not a NTFS file system (invalid sector size 0)))
```

Help me, GPT-Kenobi, you're my only hope.

> [!quote] ChatGPT
> **Check if you’re pointing at the _right layer_**
> 
> TSK works on **partitions or volumes**, not usually the whole raw disk image.
> 
> - Many forensic images include the partition table (MBR/GPT). If you point TSK directly at the disk, it may choke.
> - Use `mmls` first.
> 	- This shows partitions, their offsets, and sizes.
> - Then rerun the TSK command with `-o <offset>` (in sectors).

#### Determine partitions, their offsets, and sizes with `mmls`

```
❯ mmls recipe_usb.E01
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000000037   0000000038   Unallocated
002:  000:000   0000000038   0007839719   0007839682   NTFS / exFAT (0x07)
003:  -------   0007839720   0007856126   0000016407   Unallocated
```

#### Rerun `fls` with `-o <offset>`

```
❯ fls -r -o 0000000038 recipe_mnt/ewf1
r/r 4-128-1:    $AttrDef
r/r 8-128-2:    $BadClus
r/r 8-128-1:    $BadClus:$Bad
r/r 6-128-4:    $Bitmap
r/r 7-128-1:    $Boot
d/d 11-144-4:   $Extend
+ d/d 29-144-2: $Deleted
+ r/r 25-144-2: $ObjId:$O
+ r/r 24-144-3: $Quota:$O
+ r/r 24-144-2: $Quota:$Q
+ r/r 26-144-2: $Reparse:$R
+ d/d 27-144-2: $RmMetadata
++ r/r 28-128-4:        $Repair
++ r/r 28-128-2:        $Repair:$Config
++ d/d 31-144-2:        $Txf
++ d/d 30-144-2:        $TxfLog
+++ r/r 32-128-2:       $Tops
+++ r/r 32-128-4:       $Tops:$T
+++ r/r 33-128-1:       $TxfLog.blf
+++ r/r 34-128-1:       $TxfLogContainer00000000000000000001
+++ r/r 35-128-1:       $TxfLogContainer00000000000000000002
r/r 2-128-1:    $LogFile
r/r 0-128-6:    $MFT
r/r 1-128-1:    $MFTMirr
r/r 9-128-8:    $Secure:$SDS
r/r 9-144-11:   $Secure:$SDH
r/r 9-144-14:   $Secure:$SII
r/r 10-128-1:   $UpCase
r/r 10-128-4:   $UpCase:$Info
r/r 3-128-3:    $Volume
r/r 40-128-1:   Recipe.txt
d/d 36-144-1:   System Volume Information
+ r/r 37-128-1: IndexerVolumeGuid
+ r/r 38-128-1: WPSettings.dat
-/r * 39-128-1: do_not_open_this.txt
-/r * 41-128-3: ResultingBrunner.png
V/V 256:        $OrphanFiles
```

Nice! 

#### Check out the file `Recipe.txt`

`Recipe.txt` has the inode address of `40-128-1`. Display the contents of this file on `stdout` using `icat`:

```
❯ icat -o 0000000038 recipe_mnt/ewf1 40-128-1                                                                                  
20 g yeast                                                                                                                     
1 dl milk                                                                                                                      
40 g butter                                                                                                                    
1 egg                           
40 g sugar                      
0.5 tspn salt                   
250 g flour                     
                                                               
1. Mix it all together                                         
2. Add the secret ingredient    
3. Put it in the oven at 175 degrees for 30 minutes
4. Let it rest                                     
5. Cover the cake in marzipan and whipped cream
...                                            
6. Profit?                      
```

Sweet! No flag yet, but I'm learning about Forensics and making progress on the challenge.

#### Look at `do_not_open_this.txt`

```
❯ icat -o 0000000038 recipe_mnt/ewf1 39-128-1
C0u1d_Th1$_B3_f0r_z1p?%
```

Interesting... `C0u1d_Th1$_B3_f0r_z1p?` is not the flag, but it's certainly sweet. Worth making note of for sure. Based on the message, it could be the password to a zip archive.

## Not Solved (yet)

I didn't end up solving this one. I'm publishing the write-up in its current state anyway. May come back to fix it. May not.