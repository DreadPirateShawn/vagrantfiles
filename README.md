vagrantfiles
=================

Shared files & README for various Vagrantfile repositories.


Contents
--------
* [Setup: Required](#required) ~ *You'll need to install these to use this repo.*
* [Setup: Personalize](#personalize) ~ *Set your synced folder, share your SSH keys.*
* [Vagrant Commands](#commands) ~ *Vagrant up/destroy/etc command reference.*
* [Notes](#notes) ~ *Misc additional info, if you care.*
* [Troubleshooting](#troubleshooting) ~ *Troubleshooting.*


<a name="required"></a>
Setup: Required
-----

#### Step 1: Install VirtualBox and Vagrant

Install the appropriate packages for your system.

##### Debian ecosystem (e.g. Linux Mint)

* https://www.virtualbox.org/wiki/Linux_Downloads
  * e.g. for Linux Mint, `Linux Mint 16 ~> Ubuntu Saucy (13.10)` or `Linux Mint 17 ~> Ubuntu Trusty (14.04 LTS)`
  * *Note: On Linux Mint 17, the latest VirtualBox version is available via software manager, so use that instead. (Opening the .deb file via QApt Package Installer (right-click context menu) triggered the automatic version check and advice, so this could easily change in the future.)*
* http://docs.vagrantup.com/v2/installation/index.html
* http://www.vagrantup.com/downloads.html
  * e.g. for a Debian-based 64-bit OS, `Linux DEB 64-bit`

##### Arch ecosystem

* https://wiki.archlinux.org/title/VirtualBox
* https://wiki.archlinux.org/title/Vagrant

```sh
sudo pacman -S vagrant virtualbox virtualbox-host-modules-arch
sudo reboot
```

#### Step 2: Install vagrant plugins

* vagrant-vbguest, vagrant-share
These appear to be basic dependencies, e.g. for sharing filesystem.
* [vagrant-cachier](https://github.com/fgrehm/vagrant-cachier)
This enables apt-get to avoid re-downloading package data upon each `vagrant destroy; vagrant up;` cycle.

```sh
vagrant plugin install vagrant-vbguest vagrant-share
vagrant plugin install vagrant-cachier
```

<a name="personalize"></a>
Setup: Personalize
------------------

#### Customize synced folders

Remember to edit `config.vm.synced_folder` in Vagrantfile to personalize your synced folder path.

```
config.vm.synced_folder "/host/system/path", "/path/as/seen/from/within/vm"
```

#### Share SSH key for GitHub access

Assuming `~/.ssh/id_rsa` is your key location, this Vagrantfile is already configured to share.

```sh
Vagrant.configure("2") do |config|
  config.ssh.private_key_path = "~/.ssh/id_rsa"
  config.ssh.forward_agent = true
end
```

Your private key must be available to the local ssh-agent. On your host system, check with `ssh-add -L`, if it's not listed add with `ssh-add ~/.ssh/id_rsa`.

```sh
key_file=~/.ssh/id_rsa

# Add if not already added
[[ -z $(ssh-add -L | grep $key_file) ]] && ssh-add $key_file
```

Problems? See the [Troubleshooting](#troubleshooting) section below.



<a name="commands"></a>
Vagrant Commands
--------

| Command | What it does |
| --- | --- |
| vagrant box add ubuntu/trusty64 | Add a Vagrant box, eg Ubuntu 14.04 |
| vagrant box list | List your Vagrant boxes |
| vagrant init | Create basic Vagrantfile (not needed since you're using this GitHub project) |
| vagrant up | Run the VM for the first time |
| vagrant ssh | Use the VM |
| vagrant provision | If you've edited the Vagrantfile, this re-provisions the VM |
| vagrant suspend | Pause the VM |
| vagrant halt | Stop the VM |
| vagrant destroy | Destroy the VM |

[Official Vagrant CLI reference](http://docs.vagrantup.com/v2/cli/)

#### Alternate Vagrant boxes
* https://vagrantcloud.com/discover/popular


<a name="troubleshooting"></a>
Troubleshooting
---------------

#### Logging
```sh
VAGRANT_LOG=info vagrant ...
vagrant --debug ...
```

#### General update

This can help pre-emptively, to ensure up-to-date VM image:
```sh
vagrant box update
```

#### GitHub access from VM

A couple possible errors:

##### Error: "AMD-V is disabled in the BIOS (or by the host OS) (VERR_SVM_DISABLED)"

You may need to enable virtualization in the BIOS, e.g. something like:
* BIOS -> System Config -> Virtualization Technology <Enabled>
* BIOS -> M.I.T. -> Adv Freq Settings -> Adv CPU Settings -> SVM Mode <Enabled>

Other keywords may be VT-x, or Google around for fresh "<brand> enable virtualization" info.

##### Error: Permission denied (publickey).
##### Error: Could not open a connection to your authentication agent.

If you see one of these, then check your host system to verify whether `ssh-agent` is running. You may need to re-run it fresh, and possibly re-run the `ssh-add` commands from the [Setup: Personalize](#personalize) section above as well. (Not sure why this is, I may be overkilling the solution.)

```sh
killall ssh-agent; eval `ssh-agent`
```

##### Additional References:
* https://coderwall.com/p/p3bj2a
* https://help.github.com/articles/using-ssh-agent-forwarding
* http://stackoverflow.com/questions/11955525/how-to-use-ssh-agent-forwarding-with-vagrant-ssh
