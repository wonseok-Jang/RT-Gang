# RT-Gang: Real-Time Gang Scheduling for Safety Critical Systems
RT-Gang adds the ability to schedule one (parallel) real-time task across all cores of a multicore platform at any given time. It has been developed as a scheduling feature in the Linux kernel. This repository contains the following materials related to RT-Gang:
- Linux kernel patches for the supported platforms
  - Jetson TX-2
  - Raspberry Pi-3
  - x86
- Experiment scripts to reproduce results from our paper
- Documentation

Please note that RT-Gang is **architecture neutral** and one should be able to test it on any (reasonably recent) Linux supported platform. The *supported platforms* listed above are the ones we have officially tested RT-Gang on.

# Pre-requisites
### Hardware
One of the following platforms:
+ NVIDIA's Jetson TX-2 Board
+ Raspberry Pi-3 (Rev B)
+ Any Intel PC capable of running Linux (v4.x.x and later)

### Software
+ Linux for Tegra (Version 28.1) for Jetson TX-2
+ Raspbian Stretch with Linux (v4.4.50)
+ Python (Version 2.7)
+ Git

Supplementary software packages are needed to run experiments and analyze collected data. These are listed with the relevant experiment scripts.

# Step-by-step Instructions
1. Obtain the source of the supported Linux kernel version for the platform under test. Since this step is inherently platform / environment dependent, we leave it to the user to perform this step according to their platform of choice.

2. Patch the kernel source with the relevant RT-Gang patch from this repository.

3. Ensure that **CONFIG_SCHED_DEBUG** is enabled in the kernel configuration. Otherwise, the scheduling features won't be modifiable at runtime.

4. Build and install the kernel on the platform.

5. Once the system has rebooted, ensure that **RT_GANG_LOCK** is available as a scheduling feature in the file */sys/kernel/debug/sched_features*.
```
# Enable RT-Gang
echo RT_GANG_LOCK >> /sys/kernel/debug/sched_features

# Disable RT-Gang
echo NO_RT_GANG_LOCK >> /sys/kernel/debug/sched_features
```

## Throttling
Throttling of best-effort tasks under RT-Gang is supported with the help of BWLOCK ([code](https://github.com/wali-ku/BWLOCK-GPU), [paper](http://drops.dagstuhl.de/opus/volltexte/2018/8983/pdf/LIPIcs-ECRTS-2018-19.pdf)). To enable throttling, following steps should be performed on the platform under test:

1. Build the kernel module.
```
cd throttling/kernel_module
make
```

2. Build the user-application. This application provides interface with the kernel-module and aids the user in setting the following parameters:
  - PID of the real-time application which needs to be protected
  - BWLOCK value (1: Lock; 0: Unlock)
  - Safe memory usage threshold of corunning applications (# of LLC-misses)
```
cd throttling/user_app
make
```

3. Insert the kernel module.
```
insmod exe/bwlockmod.ko
```

4. Execute the RT-application and note its pid.

5. Execute the user-app and provide the necessary parameters as noted in Step-2.
```
# Sample RT-App: ./rt-work
# Sample Parameters: PID: 1296; Corun Threshold: 16348 (i.e., 100 MB/s in systems with 64B cache-word)
./rt-work &
[1] rt-work      1296

cd throttling/user_app
./test -p 1296 -v 1 -e 16348
```

6. BWLOCK will now ensure that all best-effort tasks are throttled @ 100 MB/s while the *rt-work* process is in execution.

# Citation
The paper for RT-Gang can be found [here](https://arxiv.org/pdf/1903.00999.pdf). Please use the following BibTex to cite it:
```
@inproceedings{wali2019rtgang,
	title = {RT-Gang: Real-Time Gang Scheduling Framework for Safety-Critical Systems},    
	author = {Waqar Ali and Heechul Yun},
	booktitle = {IEEE Intl. Conference on Real-Time and Embedded Technology and Applications Symposium (RTAS)},
	year = {2019}
}
```
