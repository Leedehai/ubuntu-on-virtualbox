<!-- Copyright (c) 2020 Leedehai. All rights reserved.
The use of this content is restricted by the CC-BY-NC-ND license, see
LICENSE.txt. -->

# Installing a headless Ubuntu On VirtualBox

> Stars are appreciated. :wink:

The work is licensed under the [CC-BY-NC-ND license](LICENSE.txt).

**Who is this guide not for**:
If you want to install a Ubuntu desktop version (with GUI) on VirtualBox, this
guide is probably unnecessarily complicated to you. Fortunately, there are many
guides out there for that topic.

**Key parameters**:
- Host OS: macOS 10.15, x86_64. But I think Windows is fine.
- Guest OS: Ubuntu 20.04 LTS, server version, x86_64.
- Virtual machine hypervisor: VirtualBox 6.1 for macOS.

**Table of contents**
- [Requirements](#requirements)
- [Let's Go](#lets-go)
    - [Download and install VirtualBox](#gift-download-and-install-virtualbox)
    - [Download Ubuntu](#gift-download-ubuntu)
    - [Create a VM](#gift-create-a-vm)
    - [Install Ubuntu on VM](#gift-install-ubuntu-on-vm)
    - [Make sure Internet is fine](#gift-make-sure-internet-is-fine)
    - [Set up SSH](#gift-set-up-ssh)
    - [Install basic programs](#gift-install-basic-programs)
    - [Install Guest Additions](#gift-install-guest-additions)
    - [Create shared folder](#gift-create-shared-folder)
    - [Set up timezone](#gift-set-time-zone)
    - [Set up time synchronization](#gift-set-up-time-synchronization)
- [Optionals](#optionals)
    - [SSH without password](#rainbow-ssh-without-password)
    - [Find Python](#rainbow-find-python)
    - [Customize welcome messages](#rainbow-customize-welcome-messages)
    - [Use ZSH and oh-my-zsh](#rainbow-use-zsh-and-oh-my-zsh)
    - [Handy sysinfo](#rainbow-handy-sysinfo)
    - [Non-default shared library path](#rainbow-non-default-shared-library-path)
    - [Add shared folder to PATH](#rainbow-add-shared-folder-to-path)

## Requirements

**Concepts**:
| concept | explanation      |
|:--------|:-----------------|
|OS       | operating system |
|VM       | virtual machine  |
|x86_64   | The common 64-bit CPU architecture on desktop machines, a.k.a. AMD64 |
|Host     | the operating system you run your virtual machine on |
|Guest    |the operating system running inside the virtual machine |

- The virtual machine is popular, (relatively) easy to use, and inexpensive.
- I can SSH into the guest Linux from my host, using it like a remote machine.
- The guest Linux should be able to network with the outside world.
- I can sync files between the host OS and the guest OS, so modifications to
  files are visible in *both* OSes. That means I can edit a program in one OS,
  and compile and run it in another.
- The guest OS should be headless, as a GUI for the guest OS bloats the binary
  size, and I rarely need a GUI as user experience with a GUI running in a
  virtual machine is unpleasant in the first place.
- Time synchronization between the host and guest should be nearly perfect. A
  slight difference in time would break tools like `make`.

## Lets Go!

> Emoji :round_pushpin: means "action item". If you are on a hurry, just follow
the action items to disregard the others.

### :gift: Download and install VirtualBox

[VirtualBox](https://www.virtualbox.org/) is an open-source free virtual machine
([manual](https://www.virtualbox.org/manual/)).

:round_pushpin: Install VirtualBox from
[the official website](https://www.virtualbox.org/wiki/Downloads), which took
around half a minute for me.

:round_pushpin: Follow the installer's GUI, which is pretty smooth. Be sure to
choose the version that suits your host OS. After installation, you should be
able to see its icon in your OS's launch pad or start menu and click it.
<img src="vbox/virtualbox_logo.png" alt="virtualbox_logo" width="25"/>

### :gift: Download Ubuntu

[Ubuntu](https://ubuntu.com/) is (one of?) the most popular open-source Linux
distributions and gateway to Linux. If you want to use niche ones like
[ArchLinux](https://www.archlinux.org/) or [Fedora](https://getfedora.org/),
you are probably not reading this newbie guide in the first place.

Ubuntu generally produces a new release twice a year, but you don't need to
upgrade that often. For general purposes, users only need Long-Term Support
(LTS) releases, which are normally released every 2 years, and each LTS release
are supported for 5 years.

Normal users download the Ubuntu Desktop version. This is the one with a Graphic
User Interface (GUI). But we don't need a GUI for reasons stated before.
Therefore, we opt for the Server version.

> Like I said in the beginning, if you want a Desktop version, there are already
many articles on how to do it.

Today the most recent LTS is Ubuntu 20.04 LTS, code name "Focal Fossa". We use
that as an example.

:round_pushpin: Download Ubuntu Server (not Desktop) from
[the official website](https://ubuntu.com/download). Be sure to choose the
**Server** version 64-bit (x86_64). Downloading took about 5 minutes on my
machine. Afterwards, you should see an ISO image file, named like
`ubuntu-20.04-live-server-amd64.iso`.

### :gift: Create a VM

:round_pushpin: Open VirtualBox, click "New", which will launch a dialog box.

> Screenshot: [vbox/new_vm_from_welcome_page.png](vbox/new_vm_from_welcome_page.png).

:round_pushpin: Page "Name and operating system". Configure it like below, and
click "Continue".
- Virtual box name: we name it `fossa`, but you can pick another one.
- Machine folder: where you want to store related file created for the VM on
  your host machine, for example `/Users/username/VBox_VMs`.
- Type: `Linux`.
- Version: `Ubuntu (64-bit)`.

> Screenshot: [vbox/create_vm_name_and_where.png](vbox/create_vm_name_and_where.png).

:round_pushpin: Page "Memory size". You may either follow its recommended
memory size, or give it more. You can adjust it after VM creation as well
(when the VM is not running) so there is no need to worry. Click "Continue".

> Screenshot: [vbox/create_vm_memory.png](vbox/create_vm_memory.png).

:round_pushpin: Page "Hard disk". Choose "Create a virtual hard disk now"
because you are creating a new VM. Click "Continue".

> Screenshot: [vbox/create_vm_disk.png](vbox/create_vm_disk.png).

:round_pushpin: Subpage "Hard disk file type". Choose the default VDI format,
  unless you want the hard disk to be portable to non-VirtualBox VMs as well.
  Click "Continue".

> Screenshot: [vbox/create_vm_disk_type.png](vbox/create_vm_disk_type.png).

:round_pushpin: Subpage "Storage on physical hard disk". Either choices are,
fine, but I chose the default "Dynamically allocated" (which is slower due to
occasional reallocation on the host), as the majority of my disk writing tasks
will be done on the shared folder (more on that later), so the writing speed for
this "native" disk is not a bottleneck. Click "Continue".

> Screenshot: [vbox/create_vm_disk_dynamic_size.png](vbox/create_vm_disk_dynamic_size.png)

:round_pushpin: Subpage "File location and size". Pick a location to store
  your virtual hard disk image, and set the size limit to be at least 10 GB
  (though Ubuntu Server is well under 5GB, you want some space for other
  programs). Click "Create".
  - If you want to store more stuff, you may pick a larger size limit. But you
    can also store the additional stuff in the shared folder so it won't occupy
    your virtual disk space.
  - You can enlarge the disk later using VirtualBox "File" > "Virtual Media
    Manager".

> Screenshot: [vbox/create_vm_disk_size_limit.png](vbox/create_vm_disk_size_limit.png).

:round_pushpin: After clicking that "Create" button, the dialog box is closed.
You should see a VM in the VirtualBox UI.

> Screenshot: [vbox/create_vm_done.png](vbox/create_vm_done.png).

### :gift: Install Ubuntu on VM

The VM you created last step is just a virtualized hardware. You need to power
it using an OS, which in our case is Ubuntu you just [downloaded](#gift-download-ubuntu).

:round_pushpin: In the VirtualBox UI, find your VM `fossa`. Click "Settings".

> Screenshot: [vbox/click_vm_settings.png](vbox/click_vm_settings.png).

:round_pushpin: In the Settings dialog, go to the tab "Storage". Insert the
Ubuntu's ISO image in the optical drive. Click "OK", closing the dialog. The
ISO image will serve as the boot medium.

> Screenshot: [vbox/select_ubuntu_image.png](vbox/select_ubuntu_image.png).

It will ask for your account name and password, and that account will be given `sudo`
access. Note it probably won't ask you for a `root` account password.

:round_pushpin: In the VirtualBox UI, find your VM `fossa`. Click "Start". This
will launch a small screen, in which you will see Ubuntu's installer guide.

> Screenshot: [vbox/click_vm_start.png](vbox/click_vm_start.png).<br>
Screenshot: [vbox/vm_installer_first_page.png](vbox/vm_installer_first_page.png).

> We can use `VBoxManage startvm fossa --type headless` to start it from host
command line, too. But for now we don't have SSH ready yet, so we need the VM
window.

:round_pushpin: Follow guides in the Ubuntu's installer. This Ubuntu release
has the following pages. Note that the username you set here will be bestowed
the `sudo` privilege.

| Page                         | Action                                                                            |
|:-----------------------------|:----------------------------------------------------------------------------------|
| Select language              | Choose your preferred language. Mine is "English". Press Enter ("Done").          |
| Installer update available   | Choose the default "Continue without updating". Press Enter ("Done").             |
| Keyboard configuration       | Just press Enter ("Done"), unless your keyboard is not the common QWERTY model.   |
| Network connections          | Just press Enter ("Done").                                                        |
| Configure proxy              | Normally you don't need to configure one, just press Enter ("Done").              |
| Ubuntu archive mirror        | Use the default URL. Press Enter ("Done").                                        |
| Guided storage configuration | Choose the default "Use an entire disk". Navigate to "Done", press Enter.         |
| Storage configuration        | Use the defaults. Press Enter ("Done").                                           |
| Confirm destructive action   | Our new virtual disk stored no data, so it's safe. Navigate to "Continue", press Enter. |
| Profile setup                | Enter your (preferred) name, server name, username (not `root`), password. Navigate to "Done", press Enter. |
| SSH setup                    | Put a cross in "Install SSH server" (because we want ssh). Navigate to "Done", press Enter. |
| Configure server snaps       | I need none of these. Navigate to "Done", press Enter. |

:round_pushpin: Wait for installing to complete. Click "Reboot". If it complains
about "Failed to unmount" the boot medium, go to VM's "Settings" where the ISO
file is loaded, unload it, and go back to the VM window, press enter.

> Screenshot: [vbox/vm_installed_want_to_reboot.png](vbox/vm_installed_want_to_reboot.png).

:round_pushpin: Wait for it to reboot. Finally, you will see a bunch of log
messages in the VM window. After the log messages stopped, press enter, and
a login prompt will be displayed below. Type your username and password, then
you are led to the shell. Type a few commands to ensure the VM is working.

> Screenshot: [vbox/vm_first_login.png](vbox/vm_first_login.png).<br>
Screenshot: [vbox/vm_first_login_after.png](vbox/vm_first_login_after.png).<br>

:round_pushpin: In the VM, check if you can switch to the `root` account:
```sh
# In VirtualBox's VM window.

su
```
If there's an error, that's because there's no `root` user on the VM (because
it didn't ask you to set one during installation). Do this:
```sh
# In VirtualBox's VM window.

sudo passwd root
# Will prompt for your password. Then, it will prompt for setting root's password.

# Check again to confirm you can switch to the root account.
su
# Prompt for root's password.
whoami
# Print: root

# Switch to your own account.
su -
whoami
# Print: your username.
```

> Screenshot: [vbox/vm_first_login_make_root.png](vbox/vm_first_login_make_root.png).

### :gift: Make sure Internet is fine

Network is necessary for the VM to communicate with the outside world (e.g.
`apt install` needs network connection to download programs). Unless you are
some kind of tech master, we will be using the Internet.

:round_pushpin: The following assumes you are able to access Google's hosts, as
they are the most stable ones. If you knew your region blocks Google domains,
try others.

```sh
# In VirtualBox's VM window.

# Try connecting to Google's DNS to confirm Internet works.
ping 8.8.8.8
# Prints network packets' meta data... which indicates OK. Ctrl+C to stop it.

# Try connecting to Google to confirm name resolution works.
ping www.google.com
# Prints network packets' meta data... which indicates OK. Ctrl+C to stop it.
```

If these commands throw errors, and you are confident your host machine is able
to reach to Internet, you probably have messed up the network configuration
during Ubuntu installation.

> Screenshot: [vbox/ensure_network.png](vbox/ensure_network.png).

### :gift: Set up SSH

It is not pleasant to work in the small window displayed by VirtualBox. You
definitely want to work in your host's terminal emulator and SSH into the VM
just like you SSH with a remote machine.

:round_pushpin: Create a host network adapter on VirtualBox.
- Click VirtualBox menu "File" > "Host Network Manager".
- Create an adapter, and manually configure the adapter **and** its DHCP server
  (see screenshots).

> Screenshot: [vbox/click_vbox_host_network_manager.png](vbox/click_vbox_host_network_manager.png).<br>
Screenshot: [vbox/create_new_adapter_configure_adapter.png](vbox/create_new_adapter_configure_adapter.png).<br>
Screenshot: [vbox/create_new_adapter_configure_dhcp.png](vbox/create_new_adapter_configure_dhcp.png).

:round_pushpin: Shut down your VM, because we are configuring a (virtual)
hardware.

```sh
# In VirtualBox's VM window.

sudo shutdown now
# The VM window closes.
```

:round_pushpin: Set the VM's network settings. Go to your VM's "Settings" >
"Network", designate a new adapter, "Adapter 2", for the VM. See screenshot
for details.

> The existing Adapter 1 is an NAT adapter, which allows your VM to communicate
with the outside world using your host's network interface. Our new adapter
(Adapter 2) is a Host-only adapter, which allows your VM to communicate with
your host.

> Screenshot: [vbox/click_vm_settings.png](vbox/click_vm_settings.png)<br>
Screenshot: [vbox/designate_new_adapter_for_vm.png](vbox/designate_new_adapter_for_vm.png)

:round_pushpin: Start your VM. Log in.

> Screenshot: [vbox/click_vm_start.png](vbox/click_vm_start.png)

:round_pushpin: Find your adapter names: your Adapter 1, Adapter 2 are mapped to
these names (in addition to a loopback interface `lo`).
```sh
# In VirtualBox's VM window.
ls /sys/class/net
# Print: enp0s3 enp0s8 lo (or: eth1 eth2 lo)
```

:round_pushpin: Configure in side your VM the network interfaces. Now we should
open the file `/etc/netplan/00-installer.yaml` (you probably want to use `sudo`
permission in order to save it) and edit it.
> Older guides might tell you to edit `/etc/network/interfaces`
([old reference](deprecated/etc_network_interfaces.md)). **Don't do that.**
Changes there won't be picked up by Ubuntu because it was
[deprecated](https://www.linuxjournal.com/content/have-plan-netplan) since
Ubuntu 18.04, unless your Ubuntu was upgraded from a previous release before
that instead of freshly installed. The new configuration system is called
[netplan](https://netplan.io/reference).
```yaml
# In VirtualBox's VM window. Open /etc/netplan/00-installer.yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: false
      addresses: [192.168.56.101/24] # This IP is in the range you configured in the Host-Only adapter
```

> Screenshot: [vbox/configure_netplan_for_ssh.png](vbox/configure_netplan_for_ssh.png)

:round_pushpin: Apply the configuration.

```sh
# In VirtualBox's VM window.

sudo netplan apply
# If there are errors: sudo netplan --debug apply
```

:round_pushpin: Let's verify it works. SSH into your VM from your host.

```sh
# In host.
VBoxManage startvm fossa --type headless # "fossa" is your VM name.
# Wait for a few seconds..
# The username was configured in "Install Ubuntu on VM" above.
# The SSH IP was configured in "Set up SSH" above.
ssh leedehai@192.168.56.101

# Yay! You are logged into the VM.
```

### :gift: Install basic programs

The Ubuntu Server is a simple distribution. It is small, but it also means you
need to install programs you need. In my case, I want `binutils` (including
`ld`, `nm`, etc.) and `build-essential` (including `libstdc++`, `gcc`, `make`).

```sh
# In VM via SSH.
sudo apt install binutils build-essential
# Wait for it to complete (less than two minutes).

# Ensure the programs work.
ld --version
make --version
```

### :gift: Install Guest Additions

VirtualBox Guest Additions provide supplemental functionalities, like the folder
sharing and time synchronization.

:round_pushpin: Log into the VM, verify you have installed deps required to run
the installer.
```sh
# In VM via SSH.
apt list build-essential dkms linux-headers-$(uname -r)

# If not, install them:
# sudo apt install build-essential dkms linux-headers-$(uname -r)
```

:round_pushpin: In the menu of the VM window (not the SSH window, but the window
that was brought up by [clicking "Start"](vbox/click_vm_start.png)), click
menu "Devices" > "Insert Guest Additions CD Image".

> Screenshot: [vbox/click_vm_devices_insert_guest_additions.png](vbox/click_vm_devices_insert_guest_additions.png)

> If you get an error saying the guest has no CD-ROM, stop
VM, in VirtualBox's UI open this VM "Settings" > "Storage", add a new CD-ROM
device to the machine by clicking on the plus sign (Adds optical device). Reboot
the VM.

:round_pushpin: Log into the VM, mount the CD, and execute the installer, which
builds some kernel modules, taking about 1 minute.

```sh
# In VM via SSH.
sudo mkdir -p /mnt/cdrom
sudo mount /dev/cdrom /mnt/cdrom
# It is fine to see WARNING: device write-protected, mounted read-only.

cd /mnt/cdrom
sudo sh ./VBoxLinuxAdditions.run --nox11

# Printout:
#
# Verifying archive integrity... All good.
# Uncompressing VirtualBox 6.1.10 Guest Additions for Linux........
# VirtualBox Guest Additions installer
# Copying additional installer modules ...
# Installing additional modules ...
# VirtualBox Guest Additions: Starting.
# VirtualBox Guest Additions: Building the VirtualBox Guest Additions kernel
# modules.  This may take a while.
# ...
# VirtualBox Guest Additions: Running kernel modules will not be replaced until
# the system is restarted
```

:round_pushpin: Reboot the VM. Login into the VM again and verify installation
was OK.
```sh
# In VM via SSH.
sudo shutdown -r now
```
```sh
# In host.
# ... reboot, login again
# Instead of starting the VM by clicking "Start", we can reboot it using
# command line in the host. This may take a few seconds before you are
# able to SSH into the VM.
VBoxManage startvm fossa --type headless

# If it says the VM is "already locked by a session (or being locked or unlocked)",
# it is probably because you forgot to close the VM window brought up by clicking
# "Start" after shutdown.
```
```sh
# In VM via SSH.
lsmod | grep vboxguest
# Print like:
# vboxguest             303104  2 vboxsf
```
If there's no printout, the guest addition is not loaded. Try installing the
guest additions one more time.

[Reference](https://linuxize.com/post/how-to-install-virtualbox-guest-additions-in-ubuntu/).

### :gift: Create shared folder

:round_pushpin: Open VM's "Settings" > "Shared Folder", configure like the
screenshot.

> Screenshot: [vbox/click_vm_settings.png](vbox/click_vm_settings.png).<br>
Screenshot: [vbox/configure_shared_folder.png](vbox/configure_shared_folder.png).

:round_pushpin: Reboot the VM, log in, you will see the shared folder is
mounted under `/media/sf_[folder name]`. If you want, you can create a symlink
with a better name.

```sh
# In VM via SSH.
ls /media
# Print: sf_Doodle ("Doodle" is my folder name, see the screenshot above)

ls /media/sf_Doodle
# Prints files under my shared folder.
```

:round_pushpin: Verify you have write access.
```sh
# In VM via SSH.

touch /media/sf_Doodle/test_file # "Doodle" is my folder name
```
If there is a permission error, that's because you haven't add your username
to the `vboxsf` group. Do this to fix that, and verify again.
```sh
# In VM via SSH.

sudo adduser $USER vboxsf

# Log out (using command "exit"), then SSH to log in again.

touch /media/sf_Doodle/test_file
# The OS no longer complains about permission. OK!
rm /media/sf_Doodle/test_file
```

### :gift: Set time zone

:round_pushpin: Set the timezone to your local time, if `date` shows the UTC
time.
```sh
# In VM via SSH.
timedatectl set-timezone America/New_York
# For help: timedatectl -h
# List all timezones: timedatectl list-timezones

# Use "date" to verify it works.
```

### :gift: Set up time synchronization

It is crucial to ensure your VM has the same time value as your host, otherwise
tools that rely on filesystem timestamps (like `make`) will break.

:round_pushpin: In your host's terminal, use these commands to config syncing:
the sync internal and threshold should be at 100 msec.
```sh
# In host.
VBoxManage guestproperty set fossa \
 "/VirtualBox/GuestAdd/VBoxService/--timesync-interval" 100
VBoxManage guestproperty set fossa \
 "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold" 100

# Verify:
VBoxManage guestproperty get fossa \
 "/VirtualBox/GuestAdd/VBoxService/--timesync-interval" # Print 100
VBoxManage guestproperty get fossa \
 "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold" # Print 100
```

You are done!

## Optionals

These actions might prove useful to your workflow.

### :rainbow: SSH without password

You can skip typing the password every time you SSH into your VM from the host.

:round_pushpin: In your host, create a key pair. When prompting for a
passphrase, DO NOT enter one. Then pass the generated public key to your VM.
```sh
# In host.
cd ~
mkdir -p .ssh
ssh-keygen -t rsa
# Printout:
#
# Generating public/private rsa key pair.
# Enter file in which to save the key (/home/your_username_for_host/.ssh/id_rsa):
# Created directory '/home/your_username_for_host/.ssh'.
# Enter passphrase (empty for no passphrase):
# Enter same passphrase again:
# Your identification has been saved in /home/your_username_for_host/.ssh/id_rsa.
# Your public key has been saved in /home/your_username_for_host/.ssh/id_rsa.pub.
# The key fingerprint is:
# 3e:4f:05:79:3a:9f:96:7c:3b:ad:e9:58:37:bc:37:e4 a@your_host

# The username was configured in "Install Ubuntu on VM" above.
# The SSH IP was configured in "Set up SSH" above.
ssh leedehai@192.168.56.101 mkdir -p .ssh
cat .ssh/id_rsa.pub | ssh leedehai@192.168.56.101 'cat >> .ssh/authorized_keys'
```

:round_pushpin: Confirm you can log into the VM without password.
```sh
# In host.
# The username was configured in "Install Ubuntu on VM" above.
# The SSH IP was configured in "Set up SSH" above.
ssh leedehai@192.168.56.101
```

[Reference](http://www.linuxproblem.org/art_9.html).

### :rainbow: Find Python

:round_pushpin: Many tools need Python to be accessible as `python`, but
Ubuntu's Python installation might be only accessible as `python3`.
```sh
# In VM via SSH.
python3 --version
# Prints something like 3.8.2
python --version
# Prints "Command not found"
```
In this case, you need to create a symlink named `python` that points to
the `python3`'s path retrieved by `which python3`.
```sh
# In VM
cd /usr/local/bin
sudo ln -s $(which python3) python
# Confirm "python" is ok.
python --version
```

You can't just create an alias named `python` because aliases are only
interpreted by your shell, not by other programs.

### :rainbow: Customize welcome messages

Welcome messages (known as "message of the day", or `motd`) are displayed
every time you log in.

Programmatic welcome messages are scripts under `/etc/update-motd.d/`. You can
edit them, or simply `chmod -x /etc/update-motd.d/scripts_you_want_to_silence`
to prevent them from running.

You can also add your static message in the file `/etc/motd` (you may need to
create it).

> Be sure to use `sudo` before your editor command and `chmod` command in order
to save the changes.

### :rainbow: Use ZSH and oh-my-zsh

The [Z Shell](https://en.wikipedia.org/wiki/Z_shell) is compatible but prettier
than [Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)), the classical
default shell, and it has more features. Together with [oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh)
config framework, it feels like great.

:round_pushpin: Get Z Shell, if it is missing.
```sh
# In VM via SSH.
which zsh
# Install Z Shell, if `which zsh` prints nothing.
sudo apt install zsh
# Set the default shell to Z Shell.
chsh -s $(which zsh)
# Switch back to Bash: chsh -s $(which bash)
```

:round_pushpin: Your shell configurations should be in `~/.zshrc`, not Bash's
`~/.bashrc`. Even if you don't have anything to add, be sure to create this
file, otherwise Z Shell will complain about it.

:round_pushpin: Log out and log in again. Use `echo $SHELL` to confirm you
are using Z Shell.

:round_pushpin: Get oh-my-zsh, by following the instructions:
[https://github.com/ohmyzsh/ohmyzsh](https://github.com/ohmyzsh/ohmyzsh). The
installation is quite smooth.

> Fun fact: The ZSH author, Paul Falstad, was [Shao Zhong](https://www.cs.yale.edu/homes/shao-zhong/)'s
student while the latter was a TA at Princeton University, and Paul thought
Shao's username, `zsh`, made a nice name for his shell project.

### :rainbow: Handy sysinfo

Program `/usr/bin/landscape-sysinfo` prints important information on your
system, like this:
```text
  System load:  0.0                Processes:               94
  Usage of /:   21.7% of 11.75GB   Users logged in:         1
  Memory usage: 2%                 IPv4 address for enp0s3: 10.0.2.15
  Swap usage:   0%                 IPv4 address for enp0s8: 192.168.56.101
```

:round_pushpin: If you want to have a handy command that invokes it, you may
create an alias in your `~/.bashrc` (or `~/.zshrc` if you use Z Shell).

```sh
# In VM via ssh. Open /etc/netplan/00-installer.yaml
alias sysinfo='/usr/bin/landscape-sysinfo'
# Note: do not put whitespaces around '='
```

:round_pushpin: Do this to allow your changes to take effect, and verify it.
```sh
# In VM via ssh.
source ~/.bashrc # or: source ~/.zshrc

sysinfo
# Prints stuff.
```

### :rainbow: Non-default shared library path

:round_pushpin: Let's say you want your programs to be able to load
`/home/username/foo/lib/libbaz.so`, so you need to add this to your `~/.bashrc`
(or `~/.zshrc` if you use Z Shell) so that your VM's program loader can find it.
```sh
# In VM via SSH. Open file ~/.bashrc or ~/.zshrc with an editor.
LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/home/username/foo/lib"
export LD_LIBRARY_PATH
```

Note `export` is important, otherwise only your shell gets the updated
environment variable but programs running from the shell don't.

> You can verify in the shell using `echo $LD_LIBRARY_PATH`.

> This step is needed if you, say, downloaded a pre-built program binary, which
needs to find its own shared libraries.

### :rainbow: Add shared folder to PATH

As mentioned before, you can choose to store stuff on the host and share them
with the VM via the [shared folder](#gift-create-shared-folder), so that you
don't need to allocate a huge disk size when [creating the VM](#gift-create-a-vm).

:round_pushpin: Let's say you want to store some executables under
`/media/sf_[folder name]/linux_bin`, and you don't want to type the full path
to invoke them, so you need to add this to your `~/.bashrc` (or `~/.zshrc` if
you use Z Shell) so that the shell can find it.

```sh
# In VM via SSH. Open file ~/.bashrc or ~/.zshrc with an editor.
PATH="$PATH:/media/sf_[folder name]/linux_bin"
```

If you want programs running from your shell to be able to find the executables
too, append `export PATH`.

> You can verify in the shell using `echo $PATH`.

■
