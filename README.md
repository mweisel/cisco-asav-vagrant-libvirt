<img alt="Vagrant" src="https://img.shields.io/badge/vagrant%20-%231563FF.svg?&style=for-the-badge&logo=vagrant&logoColor=white"/>

# Cisco ASAv Vagrant box

A procedure for creating a Cisco ASAv Vagrant box for the [libvirt](https://libvirt.org) provider.

## Prerequisites

  * [Git](https://git-scm.com)
  * [Python](https://www.python.org)
  * [Ansible](https://docs.ansible.com/ansible/latest/index.html)
  * [libvirt](https://libvirt.org) with client tools
  * [QEMU](https://www.qemu.org)
  * [Vagrant](https://www.vagrantup.com) >= 2.2.10
  * [vagrant-libvirt](https://github.com/vagrant-libvirt/vagrant-libvirt)

> Vagrant version **2.2.16** introduced a bug that *breaks* SSH connectivity - [#12344](https://github.com/hashicorp/vagrant/issues/12344)

## Steps

0\. Verify the prerequisite tools are installed.

<pre>
$ <b>which git python ansible libvirtd virsh qemu-system-x86_64 vagrant</b>
$ <b>vagrant plugin list</b>
vagrant-libvirt (0.5.1, global)
</pre>

1\. Install the `genisoimage` tool.

> Ubuntu 18.04

<pre>
$ <b>sudo apt install genisoimage</b>
</pre>

> Arch Linux

<pre>
$ <b>sudo pacman -S cdrtools</b>
</pre>

2\. Log in and download the [Cisco Adaptive Security Virtual Appliance qcow2 package for the Cisco ASAv Virtual Firewall](https://software.cisco.com/download/home/286119613/type) file. Save the file to your `Downloads` directory.

3\. Copy (and rename) the disk image file to the `/var/lib/libvirt/images` directory.

<pre>
$ <b>sudo cp $HOME/Downloads/asav9-16-1.qcow2 /var/lib/libvirt/images/cisco-asav.qcow2</b>
</pre>

4\. Modify the file ownership and permissions. Note the owner may differ between Linux distributions.

> Ubuntu 18.04

<pre>
$ <b>sudo chown libvirt-qemu:kvm /var/lib/libvirt/images/cisco-asav.qcow2</b>
$ <b>sudo chmod u+x /var/lib/libvirt/images/cisco-asav.qcow2</b>
</pre>

> Arch Linux

<pre>
$ <b>sudo chown nobody:kvm /var/lib/libvirt/images/cisco-asav.qcow2</b>
$ <b>sudo chmod u+x /var/lib/libvirt/images/cisco-asav.qcow2</b>
</pre>

5\. Clone this GitHub repo and _cd_ into the directory.

<pre>
$ <b>git clone https://github.com/mweisel/cisco-asav-vagrant-libvirt</b>
$ <b>cd cisco-asav-vagrant-libvirt</b>
</pre>

6\. Create the `day0.iso` file. The file provides the initial configuration for the Cisco ASAv. It will be mounted and read on first boot.

<pre>
$ <b>cd files</b>
$ <b>genisoimage -r -o day0.iso day0-config</b>
</pre>

7\. Copy the `day0.iso` file to the `/var/lib/libvirt/images` directory.

<pre>
$ <b>sudo cp day0.iso /var/lib/libvirt/images/</b>
</pre>

8\. Modify the file ownership and permissions.

> Ubuntu 18.04

<pre>
$ <b>sudo chown libvirt-qemu:kvm /var/lib/libvirt/images/day0.iso</b>
$ <b>sudo chmod u+x /var/lib/libvirt/images/day0.iso</b>
</pre>

> Arch Linux

<pre>
$ <b>sudo chown nobody:kvm /var/lib/libvirt/images/day0.iso</b>
$ <b>sudo chmod u+x /var/lib/libvirt/images/day0.iso</b>
</pre>

9\. Create the `boxes` directory.

<pre>
$ <b>mkdir -p $HOME/boxes</b>
</pre>

10\. Start the `vagrant-libvirt` network (if not already started).

<pre>
$ <b>virsh -c qemu:///system net-list</b>
$ <b>virsh -c qemu:///system net-start vagrant-libvirt</b>
</pre>

11\. Run the Ansible playbook.

<pre>
$ <b>cd ..</b>
$ <b>ansible-playbook main.yml</b>
</pre>

12\. Copy (and rename) the Vagrant box artifact to the `boxes` directory.

<pre>
$ <b>cp cisco-asav.box $HOME/boxes/cisco-asav-9.16.1.box</b>
</pre>

13\. Copy the box metadata file to the `boxes` directory.

<pre>
$ <b>cp ./files/cisco-asav.json $HOME/boxes/</b>
</pre>

14\. Change the current working directory to `boxes`.

<pre>
$ <b>cd $HOME/boxes</b>
</pre>

15\. Substitute the `HOME` placeholder string in the box metadata file.

<pre>
$ <b>awk '/url/{gsub(/^ */,"");print}' cisco-asav.json</b>
"url": "file://<b>HOME</b>/boxes/cisco-asav-VER.box"

$ <b>sed -i "s|HOME|${HOME}|" cisco-asav.json</b>

$ <b>awk '/url/{gsub(/^ */,"");print}' cisco-asav.json</b>
"url": "file://<b>/home/marc</b>/boxes/cisco-asav-VER.box"
</pre>

16\. Also, substitute the `VER` placeholder string with the Cisco ASA version you're using.

<pre>
$ <b>awk '/VER/{gsub(/^ */,"");print}' cisco-asav.json</b>
"version": "<b>VER</b>",
"url": "file:///home/marc/boxes/cisco-asav-<b>VER</b>.box"

$ <b>sed -i 's/VER/9.16.1/g' cisco-asav.json</b>

$ <b>awk '/\&lt;version\&gt;|url/{gsub(/^ */,"");print}' cisco-asav.json</b>
"version": "<b>9.16.1</b>",
"url": "file:///home/marc/boxes/cisco-asav-<b>9.16.1</b>.box"
</pre>

17\. Add the Vagrant box to the local inventory.

<pre>
$ <b>vagrant box add cisco-asav.json</b>
</pre>

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details
