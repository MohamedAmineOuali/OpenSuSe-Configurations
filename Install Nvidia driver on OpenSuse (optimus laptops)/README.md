How to install Nvidia Driver on OpenSuse: Optimus laptop
=======   
 
### Pre-installation Actions 

#### Take a snapshot of the system 
    $ snapper create -d beforeNvidia
#### Checks
Make sure you have no /etc/X11/xorg.conf file and no configuration files with "ServerLayout", "Device" or "Screen" sections in the /etc/X11/xorg.conf.d directory. (Clean installation fulfills that.)


### Install
#### Install NVIDIA proprietary driver
We can find Nvidia proprietary driver in many packages in OpenSuSe.

##### Nvidia driver package (if you are going to use CUDA, TensorFlow, OpenCV with Cuda.... go to the next section)
For OpenSuSE 15.1 you should add the repo `zypper addrepo --refresh https://download.nvidia.com/opensuse/leap/15.1 NVIDIA`
To select another repo for your distribution go to this link [Nvidia_drivers](https://en.opensuse.org/SDB:NVIDIA_drivers)

    $ sudo zypper addrepo --refresh https://download.nvidia.com/opensuse/leap/15.1 NVIDIA

#### Nvidia driver Cuda package
If we are going to use CUDA, TensorFlow, OpenCV, any app that uses Cuda it's better to install the drivers using the files in the Cuda repository to ensure compatibility 

For OpenSuSE 15.1 and Cuda 10.1, we can use this repository `http://developer.download.nvidia.com/compute/cuda/repos/opensuse15/x86_64/cuda-opensuse15.repo`

    $ sudo zypper addrepo http://developer.download.nvidia.com/compute/cuda/repos/opensuse15/x86_64/cuda-opensuse15.repo

To select another repo for your distribution and Cuda version use [cuda-downloads](https://developer.nvidia.com/cuda-downloads)

=> It is better to use network so that you download only the drivers


#### Install driver package
If you are using the Cuda package you should only find one version of the driver which is G04 to install it use
    
    $ sudo zypper install nvidia-glG04 x11-video-nvidiaG04 nvidia-gfxG04-kmp-default nvidia-computeG04
    
If you are using Nvidia driver package you may find multiple version of the driver you should check the appropriate version for your card

If you know what series your car, then you can use the package summary. YaST's software manager or the following commands can be used to check the available packages:

```
zypper se x11-video-nvidiaG0*
S | Name                | Summary                                                 | Type   
--+---------------------+---------------------------------------------------------+--------
  | x11-video-nvidiaG04 | NVIDIA graphics driver for GeForce 400 series and newer | package
  | x11-video-nvidiaG05 | NVIDIA graphics driver for GeForce 600 series and newer | package
```


```
zypper se -s x11-video-nvidiaG0*
S | Name                | Type    | Version     | Arch   | Repository
--+---------------------+---------+-------------+--------+-----------
  | x11-video-nvidiaG04 | package | 390.116-5.1 | x86_64 | NVIDIA    
  | x11-video-nvidiaG04 | package | 390.116-5.1 | i586   | NVIDIA    
  | x11-video-nvidiaG05 | package | 418.56-9.1  | x86_64 | NVIDIA
```
Then you can use zypper to install

    $ sudo zypper in <x11-video-nvidiaG04 or x11-video-nvidiaG05>

#### Install bbswitch
Powering off the NVIDIA card when not in use is very efficient for significantly decreasing power consumption (thus increase battery life) and temperature. However, this is complicated by the fact that the card can be powered off only when the NVIDIA kernel modules are not loaded.

`bbswitch` is the kernel module that makes it possible to power off the NVIDIA card entirely. Install it with:

    $ sudo zypper in bbswitch


#### Install Prime 
SUSE Prime is a tool used for switching between integrated Intel GPU and NVIDIA GPU on Optimus laptops. It is an alternative to Bumblebee.

With the SUSE Prime setup, all applications render either on Intel or on NVIDIA. You can switch between them using a prime-select tool. Logout and login are required for the change to take effect.

    $ zypper install suse-prime

### Config
####Blacklist the NVIDIA modules so it can be loaded only when necessary

NOTE: Configuration should be done before rebooting to prevent any potential problem like back screen

The NVIDIA openSUSE package adds the NVIDIA driver modules to the kernel initrd image. This will make the system always load them on boot. This is problematic for disabling the NVIDIA card with `bbswitch` as it can only turn off the card when the modules are not loaded. Instead of unloading the modules before making use of bbswitch, the reverse is way easier: have the NVIDIA modules always unloaded and load them only when needed. To prevent the modules from being automatically loaded on boot, we need to blacklist them in initrd. This is easily done with:


    $ move 09-nvidia-blacklist.conf to /etc/modprobe.d/

Now we run this command `sudo dracut -f` to take the blacklist into consideration 


This will also blacklist the nouveau module which can really get in the way with Optimus and causing black screens.

NOTE: nouveau is an open-source driver for NVIDIA. There are two drivers for NVIDIA:  
* NVIDIA proprietary driver 
* nouveau developed by the community 

&#8594; For this tutorial we are not using nouveau so it should not be loaded automatically 


### Usage

#### Use the GPU for specific apps that use CUDA (like TensorFlow, OpenCV)
You can use the script file `gpu-nvidia` to start or turn off you GPU

&#8594; Note: You should add execution permission to be able to execute the script after download
   
    $ chmod u+x gpu-nvidia
    
To turn on 

    $ sudo gpu-nvidia start
   
To turn off

    $ sudo gpu-nvidia stop
    
you can also use these commands 

    $ sudo tee /proc/acpi/bbswitch <<<ON && sudo modprobe nvidia_uvm && echo "GPU is ON"
    
The first command turn on the Nvidia card the second load the library

    $ sudo rmmod nvidia_drm nvidia_modeset nvidia_uvm nvidia && sudo tee /proc/acpi/bbswitch <<<OFF && echo "GPU is OFF"
    
The first command unloads the library the second turn off the Nvidia GPU


#### GPU for GUI app 
Run KDE or Genom on NVIDIA which will run any graphical app or Cuda app on the GPU

To switch between Intel and NVIDIA, run as root:

    $ prime-select nvidia
or

    $ prime-select intel

Then logout and login to apply the changes.


#### verification 
To check if the NVIDIA GPU is on and the libraries are loaded 

    $ nvidia-smi
    
This command should output text similar to this

```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 418.87.00    Driver Version: 418.87.00    CUDA Version: 10.1     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 950M    Off  | 00000000:01:00.0 Off |                  N/A |
| N/A   57C    P0    N/A /  N/A |      0MiB /  4046MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```
It means that your GPU is running and library are loaded 

If you are using prime to run your GPU you should see KDE/Genom process running in the Process section

```

```
### Source
* [SUSEPrime](https://github.com/openSUSE/SUSEPrime)
* [archlinux bumblebee](https://wiki.archlinux.org/index.php/bumblebee#CUDA_without_Bumblebee)
* [NVIDIA_SUSE_Prime](https://en.opensuse.org/SDB:NVIDIA_SUSE_Prime)
* [cudarun](https://gitlab.com/Queuecumber/cudarun/blob/master/cudarun)
* [NVIDIA_drivers](https://en.opensuse.org/SDB:NVIDIA_drivers)
## Contributors 
 
| [<img src="https://avatars1.githubusercontent.com/u/20454717?s=460&v=4" width="100px;"/><br /><sub><b>Mohmaed Amine Ouali</b></sub>](https://github.com/MohamedAmineOuali)<br /> 
| :---: |  
