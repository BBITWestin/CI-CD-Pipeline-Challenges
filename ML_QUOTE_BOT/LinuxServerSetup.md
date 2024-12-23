# Steps to recreate

Machine 1 will be the windows machine you are using to follow these steps.
Machine 2 will be the machine you plan on setting the server up on.

### BOOT DRIVE

1. Plug in an empty USB to machine 1.
2. Download [LTS Ubuntu Server](https://ubuntu.com/download/server)
3. Download and open [Rufus](https://rufus.ie/en/)
4. Select the correct drive and iso file then keep default settings and click
   start. (this will take a few minutes).
5. When it's done eject the hard drive and plug it into machine 2.

### Linux Boot

1. Plug in ethernet to machine 2 before going any further.
2. Restart machine 2 and enter the bios to update the boot sequence. It will be
   something like "UEIF: blah blah random string". Apply changes and restart
   without entering the bios.
3. Wait for it to boot up..
4. Enter "English"
5. Enter "Done" for keyboard layout
6. Enter "Done" for "Ubuntu Server"
7. Enter "Done" for Network config (as long as you see the DHCPv4 ip)
8. Enter "Done" for Proxy (not needed)
9. Let the Ubuntu archine mirror config do its thing and then Enter "Done" (i
   pressed enter when i saw "Reading package list...")
10. For the next two pages follow this
    [guide](https://discourse.ubuntu.com/t/configuring-storage-in-the-server-installer/16691).
    Or just keep defaults, defaults are prob fine.
11. Finish the install process. ...

### Setup Linux Server

12. After the installation is complete and you've rebooted, log in with the
    credentials you set during installation.

13. Update the package list and upgrade installed packages:

    ```
    sudo apt update && sudo apt upgrade -y
    sudo apt install openssh-client openssh-server
    ```

14. After installation,
    [start and enable the SSH service](https://www.geeksforgeeks.org/ssh-command-in-linux-with-examples/):

    ```
    sudo systemctl status ssh
    ```

    ```
    sudo systemctl start ssh
    sudo systemctl enable ssh
    ```

15. Set up SSH for secure remote access:

    a. On Machine 2 (Linux server)

    ```
    chmod 700 ~/.ssh
    ```

    b. On Machine 1 (Windows), open Windows Terminal and use the following
    commands:

    - Assuming your flash drive is assigned the letter D:
    - If it's different, replace D: with the correct drive letter

    ```
    # Copy the public key to a text file on the flash drive
    type C:\Users\TrevorWestin\.ssh\id_ed25519.pub > D:\ssh_public_key.txt
    ```

    c. Safely eject the flash drive from Machine 1 and insert it into Machine 2.

    d. On Machine 2, mount the flash drive and copy the key:

    ```
    # Create a mount point
    sudo mkdir /mnt/usb

    # Mount the flash drive (replace sda1 with the correct device, use lsblk to find it)
    sudo mount /dev/sda1 /mnt/usb

    # Copy the public key to authorized_keys
    cat /mnt/usb/ssh_public_key.txt >> ~/.ssh/authorized_keys

    # Set correct permissions
    chmod 600 ~/.ssh/authorized_keys

    # Unmount the flash drive
    sudo umount /mnt/usb
    ```

    e. You can now remove the flash drive from Machine 2.

    f. Check the authorized keys file to see if it worked.

    ```
    cat ~/.ssh/authorized_keys
    ```

16. Configure SSH to disable password authentication (optional but recommended):

    a. Edit the SSH configuration file:

    ```
    sudo nano /etc/ssh/sshd_config
    ```

    b. Find and modify or add/uncomment these lines:

    ```
    MaxSessions 10
    PubkeyAuthentication yes
    AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2
    PasswordAuthentication no
    ```

    c. Save and exit the file.

    d. Restart the SSH service:

    ```
    sudo systemctl restart ssh
    ```

17. Find the IP address of Machine 2 (Linux server): a. On Machine 2, run the
    following command:

    ```
    ip addr show
    ```

    or

    ```
    hostname -I
    ```

    b. Look for the IPv4 address associated with your network interface (usually
    eth0 for wired connections or wlan0 for wireless). It will typically start
    with 192.168, 10., or 172.

    c. Make note of this IP address as you'll need it to connect from Machine 1.

18. Test your SSH connection from Machine 1 (Windows):

    ```
    ssh username@Machine_2_IP
    ```

    Replace 'username' with your actual username on the Linux server and
    'Machine_2_IP' with the IP address you found in step 17.

19. Install Git, curl, Python3, pip, and venv:

    ```
    sudo apt install -y curl git python3 python3-pip python3-venv
    ```

    - [Github CLI](https://github.com/cli/cli/blob/trunk/docs/install_linux.md)

20. Verify the installations:

    ```
    echo "Git version:"
    git --version
    echo "GitHub CLI version:"
    gh --version
    echo "Python version:"
    python3 --version
    echo "pip version:"
    pip3 --version
    ```

21. Log into Github:
    ```
    gh auth login
    ```

### Configure BIOS for Auto-Reboot After Power Failure

1. Restart your linux machine (Machine 2).

2. As the computer is booting up, repeatedly press the F2 key to enter the BIOS
   setup.

3. Once in the BIOS setup, use the arrow keys to navigate through the menus.

4. Look for a section called "Power Management", "Power Options", or something
   similar.

5. Within this section, find an option labeled "AC Recovery", "After Power
   Loss", or "Auto Power On".

6. Change this setting to "Power On" or "Last State". Here's what these options
   typically mean:

   - Power On: The system will always turn on when power is restored.
   - Last State: The system will return to whatever state it was in when the
     power was lost.
   - Power Off: The system will remain off when power is restored (this is
     usually the default).

7. After making this change, look for an option to save and exit. It's often
   done by pressing the F10 key.

8. Confirm that you want to save the changes and exit.

9. The computer will restart with the new settings applied.
