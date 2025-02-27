
##  Step-by-Step Guide

###  Step 1: Start Fresh and Set Up the Working Directory

`mkdir tinycore-remaster`

`cd tinycore-remaster`

**Description**:
- mkdir tinycore-remaster creates a new directory for your work, and cd tinycore-remaster moves into it. This directory will hold all files for modifying the ISO.
  

---

### Step 2: Create Initial Structure with Proper Permissions

`mkdir mnt new-iso extract`

```sudo chown -R $USER:$USER ./*``` 

 Download the CorePure64 ISO 

`wget -O CorePure64.iso http://www.tinycorelinux.net/14.x/x86_64/release/CorePure64-14.0.iso`


**Description**:

- mkdir mnt new-iso extract creates three directories for mounting the ISO, building the new ISO, and extracting files, respectively.

- sudo chown -R $USER:$USER ./* changes ownership of all files and directories in tinycore-remaster to your WSL user, ensuring you have proper permissions to work without constant sudo. This avoids permission errors in WSL.
  
- wget -O CorePure64.iso http://www.tinycorelinux.net/14.x/x86_64/release/CorePure64-14.0.iso downloads the CorePure64-14.0.iso (CLI-only, 64-bit) from the Tiny Core website, saving it as CorePure64.iso in your current directory.
  

---

### Step 3: Mount and Prepare the Working Environment


 Mount the ISO

`sudo mount -o loop CorePure64.iso mnt/` 

Copy contents with proper permissions 

`sudo cp -r mnt/* new-iso/`

`cd new-iso/boot`

Create and extract to temp directory

`sudo mkdir -p temp`

`cd temp`

`sudo zcat ../corepure64.gz | sudo cpio -i -H newc -d`


Create TCE directory structure

`sudo mkdir -p tce/optional`

  

**Description**:

- sudo mount -o loop CorePure64.iso mnt/ mounts the ISO file as a read-only filesystem in the mnt directory using a loop device, allowing you to access its contents. sudo is needed because mounting requires root privileges in WSL.
  
- sudo cp -r mnt/* new-iso/ copies all files from the mounted ISO to new-iso, preserving the directory structure. The -r flag ensures recursive copying, and sudo ensures proper permissions.
  
- cd new-iso/boot navigates to the boot directory where the kernel (vmlinuz) and initramfs (corepure64.gz) reside.
  
- sudo mkdir -p temp creates a temp directory for extracting files, with sudo for proper permissions.
  
- cd temp moves into the temp directory.
  
- sudo zcat ../corepure64.gz | sudo cpio -i -H newc -d extracts the corepure64.gz initramfs file (Tiny Core’s RAM-based filesystem) into temp. zcat decompresses the gzip file, and cpio unpacks it using the newc format, creating directories as needed (-d).
  
- sudo mkdir -p tce/optional creates the tce/optional directory structure for TCZ extensions, with sudo ensuring correct permissions for later use.
  

---

### Step 4: Download and Prepare Necessary Packages


 Go to optional directory cd tce/optional # Download all required packages 

`sudo wget http://www.tinycorelinux.net/14.x/x86_64/tcz/nano.tcz`   
`sudo wget http://www.tinycorelinux.net/14.x/x86_64/tcz/ncursesw.tcz`   
`sudo wget http://www.tinycorelinux.net/14.x/x86_64/tcz/file.tcz`   
`sudo wget http://www.tinycorelinux.net/14.x/x86_64/tcz/liblzma.tcz`   
`sudo wget http://www.tinycorelinux.net/14.x/x86_64/tcz/xz.tcz`   
`sudo wget http://www.tinycorelinux.net/14.x/x86_64/tcz/bzip2-lib.tcz`   

Create onboot.lst with proper permissions 

`sudo bash -c 'cat > ../onboot.lst << EOL nano.tcz ncursesw.tcz file.tcz xz.tcz liblzma.tcz bzip2-lib.tcz EOL'`

  

**Description**:

- cd tce/optional navigates to the tce/optional directory where TCZ extensions will be stored.
  
- The sudo wget commands download pre-compiled TCZ extensions from the Tiny Core repository for version 14.0 (x86_64). These include nano.tcz (text editor), ncursesw.tcz (library for terminal apps), file.tcz (file type checker), liblzma.tcz and xz.tcz (compression tools), and bzip2-lib.tcz (compression library). sudo ensures downloads have proper permissions.
  
- sudo bash -c 'cat > ../onboot.lst << EOL ... EOL' creates a file onboot.lst in the parent tce directory, listing the TCZ extensions to load automatically on boot. This uses a heredoc (EOL) to write the list with proper permissions, ensuring Tiny Core mounts these extensions during startup.
  

---

### Step 5: Extract and Integrate Packages

  
Create extract directory and extract packages 

`for f in *.tcz; do sudo mkdir -p "../../extract/$f" sudo unsquashfs -f -d "../../extract/$f" "$f" done` 

`# Copy files to main filesystem 

`cd ../../extract` 

`sudo cp -r */usr/* ../usr/` 

`cd ..`

  

**Description**:

- for f in *.tcz; do ... done loops through all .tcz files in tce/optional, extracting each one using unsquashfs (part of squashfs-tools).
  
- sudo mkdir -p "../../extract/$f" creates a directory for each TCZ file in extract with proper permissions, using the TCZ filename (e.g., nano.tcz).
  
- sudo unsquashfs -f -d "../../extract/$f" "$f" extracts the contents of each TCZ file into its corresponding directory, overwriting any existing files (-f). This unpacks Squashfs archives into a readable filesystem structure.
  
- cd ../../extract navigates to the extract directory containing unpacked TCZ contents.
  
- sudo cp -r */usr/* ../usr/ copies all files under usr/ from each unpacked TCZ to the usr/ directory in the initramfs (in extract), ensuring the extensions’ files (e.g., nano, libraries) are integrated into the system. sudo ensures permissions match.
  
- cd .. returns to the extract parent directory.
  

---

### Step 6: Create New Compressed Filesystem
  

`# Pack everything with proper permissions 

`sudo bash -c 'find . | cpio -o -H newc | gzip -9 > ../corepure64.gz.new` 

`cd .. `

`sudo mv corepure64.gz.new corepure64.gz`

  

**Description**:

- sudo bash -c 'find . | cpio -o -H newc | gzip -9 > ../corepure64.gz.new' rebuilds the initramfs by finding all files in extract, packing them into a cpio archive with the newc format, compressing it with gzip at maximum compression level (-9), and saving it as corepure64.gz.new in the parent boot/temp directory. sudo ensures proper permissions for the output file.
  
- cd .. moves back to the boot/temp directory.
  
- sudo mv corepure64.gz.new corepure64.gz renames the new initramfs file to replace the original corepure64.gz, maintaining the expected filename for Tiny Core’s boot process.
  

---

### Step 7: Create Directory Structure for the ISO

`# Go back to new-iso directory 
`cd ~/tinycore-remaster/new-iso 

`# Create TCE structure in final ISO 
`sudo mkdir -p boot/tce/optional   
`sudo cp boot/temp/tce/optional/*.tcz boot/tce/optional/   
`sudo cp boot/temp/tce/onboot.lst boot/tce/`

  

**Description**:

- cd ~/tinycore-remaster/new-iso returns to the new-iso directory where the ISO structure is being built.
  
- sudo mkdir -p boot/tce/optional creates the tce/optional directory in the ISO’s boot directory, where TCZ extensions will reside, with sudo for proper permissions.
  
- sudo cp boot/temp/tce/optional/*.tcz boot/tce/optional/ copies all downloaded TCZ files from boot/temp/tce/optional to boot/tce/optional in the ISO structure, ensuring they’re included for booting.
  
- sudo cp boot/temp/tce/onboot.lst boot/tce/ copies the onboot.lst file to boot/tce/, specifying which extensions load on boot.
  

---

### Step 8: Create the Final ISO

  

Go back to main directory 

`cd ~/tinycore-remaster` 

Create ISO with proper permissions 

`sudo genisoimage -l -J -R -V "TC-NANO" -no-emul-boot -boot-load-size 4 \ -boot-info-table -b boot/isolinux/isolinux.bin \   -c boot/isolinux/boot.cat -o CorePure64-with-nano.iso new-iso/`


**Description**:

- cd ~/tinycore-remaster navigates back to the main working directory.
  
- sudo genisoimage ... creates a new ISO file (CorePure64-with-nano.iso) from the new-iso directory using genisoimage (or mkisofs, if preferred). The options specify:  
    - -l: Allow long filenames.
      
    - -J: Create Joliet extensions for Windows compatibility.
      
    - -R: Use Rock Ridge extensions for Unix filesystems.
      
    - -V "TC-NANO": Set the ISO volume label to “TC-NANO”.
      
    - -no-emul-boot -boot-load-size 4 -boot-info-table: Configure bootable CD settings for Tiny Core.
      
    - -b boot/isolinux/isolinux.bin: Specify the bootloader (Syslinux).
      
    - -c boot/isolinux/boot.cat: Include the boot catalog.
      
    - -o CorePure64-with-nano.iso: Output the new ISO file.
  
- sudo ensures proper permissions for creating the ISO.
  

---

### Step 9: Add a Custom C++ Binary (bsl) to /usr/local/bin

  

**Command** (to be inserted after Step 5, before rebuilding the initramfs):

 ompile your C++ binary (in WSL/Ubuntu) 
 
`cd ~/tinycore-remaster`  

Add the binary to the initramfs 

`sudo mkdir -p extract/usr/local/bin` 

`sudo cp <binary_file> extract/usr/local/bin/`

`sudo chmod +x extract/usr/local/bin/<name>`


**Note**: Insert these commands after Step 5 (before sudo cp -r */usr/* ../usr/) to include binary_file in the initramfs. Then, proceed with Steps 6–8 to rebuild and create the ISO.

  

---

### Step 10: Test the Modified ISO with QEMU


`qemu-system-x86_64 \ -m 512M \   -cdrom CorePure64-with-nano.iso \   -boot d` 

**Description**:

- qemu-system-x86_64 launches the QEMU emulator for x86_64 architecture, installed on WSL or Windows.
  
- -m 512M allocates 512 MB of RAM for the virtual machine, sufficient for Tiny Core CorePure64.
  
- -cdrom CorePure64-with-nano.iso boots from your newly created ISO as a CD-ROM.
  
- -boot d specifies booting from the CD-ROM drive.
  
- -nographic runs QEMU in text-only mode, matching CorePure64’s CLI-only nature and avoiding a graphical interface.
  
- This tests your ISO in a virtual machine, verifying the CLI, extensions (nano, etc.), and custom binary (bbbbb) work as expected.
  

---

### Step 11: Copy the ISO to Windows (Optional)

List Windows users to find correct username 

`ls /mnt/c/Users/`


Copy to Windows Downloads (replace HP with your actual Windows username) 

`cp CorePure64-with-nano.iso /mnt/c/Users/HP/Downloads/`

  
**Description**:

- ls /mnt/c/Users/ lists Windows user directories in WSL to find your Windows username (e.g., “HP”).
  
- cp CorePure64-with-nano.iso /mnt/c/Users/HP/Downloads/ copies the modified ISO to your Windows Downloads folder via WSL’s /mnt/c/ mount point, allowing you to access it in Windows for further use or sharing.
  

---

## Key Takeaways

- You’ve created a custom Tiny Core Linux ISO with CLI-only functionality, adding nano and other tools, plus your  binary.
  
  

  


