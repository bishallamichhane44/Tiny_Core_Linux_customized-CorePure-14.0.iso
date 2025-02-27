# Modifying the Tiny Core Linux to create a custom remastered ISO of Tiny Core Linux


## Overview

**This project was completed as part of a 6th-semester Operating System course project.** 

Here, we modified the Tiny Core Linux CorePure64-14.0.iso (CLI-only, 64-bit version) to include additional TCZ extensions (e.g., nano, ncursesw, etc.) and a custom C++ binary (`bsl`) in `/usr/local/bin`. The resulting ISO (`CorePure64-with-nano.iso`) retains Tiny Core’s minimal, RAM-based, CLI-only nature, demonstrating concepts like lightweight OS design, boot processes, and filesystem customization. For this process, I used Ubuntu running in Windows Subsystem for Linux (WSL) on Windows 11, and tested the modified ISO in QEMU for verification. This manual assumes you’re a beginner and includes detailed explanations for each step, focusing on proper permissions and paths.



---


**Objective**
Choose any OS , analyze it and  add/implement new features in it.
  
**What we have achieved**
 1. Added the nano text editor
 2. Added the bsl showtree program as an alternative of tree command
 3. Added a hangman game.


**Methods of modifying the OS to get updated iso file**
- **Our Approach**:  
    - We started with the pre-built CorePure64-14.0.iso, extracted its initramfs (corepure64.gz), added pre-compiled TCZ extensions and custom binary (written in C++), and repacked the initramfs into a new ISO.
      
    - This is a _remastering_ process (as described in Chapters 11–13 of book "Into the Core"), focusing on modifying the existing filesystem and ISO structure without compiling source code.
      
    - It’s quicker and requires less development expertise but is limited to working with existing binaries and scripts.
    
    **Benefits**:  
	
	1. **Speed and Simplicity**: Remastering is faster and easier than compiling source code. You don’t need a full development environment or deep knowledge of kernel compilation—just basic Linux skills and tools like cpio, gzip, and genisoimage.
	  
	2. **Minimal Resources**: It works well on Tiny Core itself or a lightweight Linux system, aligning with Tiny Core’s philosophy of minimalism.
	  
	3. **Preserves Stability**: You’re modifying a working, tested system (CorePure64-14.0), reducing the risk of breaking core functionality compared to compiling from source.
	  
	4. **Focus on Extensions**: Adding TCZ extensions (like nano) and a custom binary is straightforward, meeting your goal of including custom binary in /usr/local/bin without rebuilding the entire OS.
	  
	5. **Suitable for CLI-Only**: CorePure.iso’s simplicity makes remastering ideal for adding CLI tools without introducing GUI dependencies.


---
## Prerequisites

- **Operating System**: Ubuntu installed via WSL on Windows 11.
- **Tools**:  
    - Ensure these are installed on Ubuntu WSL:  
        `sudo apt update sudo apt install -y wget squashfs-tools genisoimage build-essential`
- **QEMU**: Installed on  WSL for testing the ISO.  
    - Install QEMU on WSL:  
        `sudo apt install -y qemu-system-x86`
- **Internet Connection**: Required to download the Tiny Core ISO and TCZ files.
- **Windows Username**: Needed to copy the final ISO to Windows (verify with ls /mnt/c/Users/).
  

---
