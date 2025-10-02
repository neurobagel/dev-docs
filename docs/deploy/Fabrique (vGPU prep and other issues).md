---
title: Fabrique (vGPU prep and other issues)
tags: [Infrastructure]

---

1st attempt
After setting up the machine and adding collaborators ssh keys, we attempted to prepare the vGPU following the CC instructions here:
https://docs.alliancecan.ca/wiki/Using_cloud_vGPUs#Preparation_of_a_VM_running_Ubuntu22

The installation of packages failed when running the post-installation script for nvidia-vpgu-kmod package.
Running `nvidia-smi` command outputs:
`NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver. Make sure that the latest NVIDIA driver is installed and running.
`
Reached out to support for assistance and they suggested starting from a fresh instance.

2nd attempt
Shut off the instance, deleted the instance, and started a fresh instance. Once the machine came up, followed the CC instructions to prepare the vGPU. This time installation was successful and `nvidia-smi` command output was as outlined in the instructions. Set up the machine (docker, ssh keys, etc). Created a snapshot of the machine at this point where `nvidia-smi` was still working as expected.
Then installed Python3-venv, Python3-pip, and NVIDIA container toolkit  which is required to use Ollama containers following the instructions here:
https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#installation

and realized the `nvidia-smi` is no longer working. `nvidia-smi` command output: `Failed to initialize NVML: Unknown Error`         

3rd attempt
We rebuilt the machine using the snapshot and then installed python3-venv, python3-pip, and NVIDIA container toolkit and `nvidia-smi` command output was as outlined in the CC instructions and working as expected. However, later that day somehow the `nvidia-smi` command stopped working as expected giving `Failed to initialize NVML: Unknown Error` as output

4th attempt
Rebuilt the machine using the snapshot again and installed NVIDIA container toolkit and `nvidia-smi` command output was as outlined in the CC instructions and working as expected.

*sidenode*: Support got back to us mentioning there may be an error due to a mismatch between the Nvidia driver and the Nvidia cuda toolkit. They suggested trying version 11 of the toolkit instead. That has been installed as well, waiting to see whether the machine will break or not.
It seems the installations were for cuda toolkit version 11.8 but the `nvidia-smi` command output indicates that the machine seems to be using the old 11.4 as opposed to the newly installed 11.8.

1st update on the 4th attempt:
A week has passed and the `nvidia-smi` command output is still as expected.
- Raya has tried the machine and hasn't had any issues. 
- James mentioned what he was trying on the machine was related to hugging face and not Ollama.
*Waiting on Brent to get back to me to see if he had any issues using the machine. Waiting on Barbara and Tvisha to try the machine for more info

2nd update on the 4th attempt:
It's been a couple of weeks since the last update. The machine seems to be stable. Collaborators have been using it and it hasn't had the previously mentioend issues. The `nvidia-smi` command runs as expected and the nvidia container toolkit seems to be working fine.
The only reported issue from collaborators in the past week (first week of July 2024) seems to be the speed of the machine (to be mentioned to support).

July 9th update:
We were given notice through email:
>Hello,
the hypervisor your VM is running on, will be upgraded to the latest version
of vGPU drivers on:
Tuesday, 9th July 2024
VM name: fabrique
VM ID: 75b9d54a-6b2d-4477-8f89-d8a453557186
The 470.x driver installed within your VM will stop working and needs to be upgrade
to the 550.x version we provide.
Instructions on how to install the new driver version can be found here:
https://docs.alliancecan.ca/wiki/Using_cloud_vGPUs
Please make sure to fully uninstall the 470 version.
The following OS versions are EOL and are not supported anymore:
CentOS7
Ubuntu18
Debian10
If you need help with the driver installation within the VM, please respond to this ticket.
Thank you for your understanding.

The `nvidia-smi` command stopped working as expected and now outputs ```No devices were found```.
Drivers were updated following the CC instructions and everything seems to be working as expected.
It's worth noting that the `nvidia-smi` command now outputs the new versions of drivers specifically 550 for nvidia-smi and driver version and 12.4 for CUDA version.