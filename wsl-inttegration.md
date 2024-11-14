# WSL Inttegration

Here's a consolidated guide to setting up your WSL environment for building and running CertiKOS, including package installations and necessary changes after a fresh install.

```bash
# Update package list
sudo apt update

# Install essential packages
sudo apt install -y build-essential gcc-multilib qemu-system-x86 gdb parted python3 python3-pip
```

#### Steps After Installation

1. **Edit `make_image.py`**:
   *   Ensure the shebang line at the top of `make_image.py` is updated to use Python 3:

       ```python
       #!/usr/bin/env python3
       ```
   *   If `make_image.py` was originally created on Windows, convert it to Unix-style line endings:

       ```bash
       dos2unix make_image.py
       ```
2. **Update the Makefile to Call `make_image.py` with the Correct Path**:
   *   Open the `Makefile` and find any instance of `make_image.py`. Make sure it is called with the correct relative path:

       ```makefile
       ./make_image.py
       ```
3. **Compile the Project**:
   *   Run the following commands to clean, compile, and test:

       ```bash
       make clean
       make
       make TEST=1
       ```
4. **Run `make_image.py` Manually (if needed)**:
   *   If `make_image.py` does not run automatically, execute it directly to generate the disk image:

       ```bash
       ./make_image.py
       ```
5. **Run CertiKOS with QEMU**:
   *   Start the CertiKOS kernel using QEMU:

       ```bash
       qemu-system-x86_64 -drive file=certikos.img,format=raw
       ```

Following these steps should set up and execute CertiKOS in WSL. Let me know if you encounter any issues!
