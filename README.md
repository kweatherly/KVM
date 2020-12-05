# Checking the host machine for compatibility

The first step is to setup a suitable host machine. It's recommended to have at least 2GB or RAM and a 64-bit host system.
For our purposes we will use a host system running Ubuntu 20.04 LTS with 4GB Ram.

You need to make sure your host supports KVM virtualization and needs to have either a an Intel processor with the VT-x (vmx) or
an AMD processor with the AMD-V (svm). Run the following command to check for this;

    grep -Eoc '(vmx|svm)' /proc/cpuinfo

 If this command outputs a number greater than zero you are good to go. This output is the number of the CPU cores available for
 virtualization. If you see zero output then your host does not support virtualization.
 
 Next we need to check to see if the host system can run hardware-accelerated KVM virtual machines. Run the following command to check for this;
 
    kvm-ok
     
 If everything is good you will see a message like this;
 
    INFO: /dev/kvm exists
    KVM acceleration can be used
     
 You might need to install the package cpu-checker if the command isn't available. You can do that be running this command;
 
    sudo apt install cpu-checker
 
 # Install KVM and virtualization packages
 
    sudo apt install qemu-kvm libvirt-bin bridge-utils virtinst virt-manager
 
 After this install the libvirt daemon will start automagically. You can make sure it's running using this command;
     
    sudo systemctl is-active libvirtd
 
 Add the user who will create and manage the KVM virtual machines (Note $USER will be the current logged in user)
 
    sudo usermod -aG libvirt $USER
    sudo usermod -aG kvm $USER
 
 # Create a virtual machine
 
 Now we can actually create our virtual machine. We are going to create a CentOS 8 VM. We require the ISO image of the OS
 which we can download from here:  http://isoredirect.centos.org/centos/8/isos/x86_64/
 
 We used the minimal ISO of CentOS 8 as it'll be running on a server so we don't want a GUI
 
 One the ISO is downloaded copy it to the following folder on your host machine;
 
     /var/lib/libvirt/boot/
 
 Now we run the virt-install to actually install the VM on the host machine. The following command sets up the VM with 2 vCPUs
 and 8GB RAM. We are using the default network bridge called virbr0 which was setup when installing KVM and the virtualization packages.
 
    export ISO="/var/lib/libvirt/boot/CentOS-8.2.2004-x86_64-minimal.iso"
    export IMG="/var/lib/libvirt/images/centos8.qcow2"

    sudo virt-install --virt-type=kvm --name centos8.2 --ram 8192 --vcpus=2 --os-type=linux --os-variant=generic --virt-type=kvm --hvm --cdrom=${ISO} --network=bridge=virbr0,model=virtio --graphics vnc --disk path=${IMG},size=40,bus=virtio,format=qcow2

This step might take some time to complete but once it's done your VM will be running. VNC is automatically setup when running the KVM guest.
Hopefully you are behind a firewall of some type for secuirty reasons so the VNC port will not be exposed to the open internet. An easy way to
access the machine in this case is to use a SSH Tunnel. On a client machine that you wish to use to connect to the server run the command;

    ssh me@myserver -L 5900:127.0.0.1:5900
    
This will tunnel VNC traffic over a secure SSH connection. Now you can open the following link in your favorite VNC client;

    127.0.0.1:5900
    
That's it. Enjoy your new VM!
