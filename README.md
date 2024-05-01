# NVIDIA-Driver390-Setup #
This documentation walks you through the installation of NVIDIA driver version 390.157 and CUDA-Toolkit-5.0 on Ubuntu 22.04 (kernel 6.2.0-39-generic). At the end of this guide you should be able to render the result of nbody sample from CUDA-Toolkit.

Note: During this documentation, feel free to `man page` every command you're not sure of

## Installing NVIDIA driver ##
It all began when I upgraded my Ubuntu kernel version from `6.2.0-39-generic` to `6.5.0-generic` and my NVIDIA graphic card stopped working properly. 

You can issue the kernel version via
```bash
$ uname -r
```
```console
6.2.0-39-generic
```
If you just want a short name of your system, normally your OS name you can do it via
```bash    
$ uname -m
```
```console
Linux
```
A full system description can be issued via

    $ uname -a

On Ubuntu when you click Power button\Settings\Info your Graphic Setting should give you detail like
```console
GeForce 605/PCIe/SSE2
```
and not
```console
llvmpipe
```
When the latter renderer is used which normally means the NVIDIA is not working properly it gives you very poor graphic quality which discourages you to use your machine a lot 🤣 Asking Microsoft Bing Copilot led me to [this ubuntuforum thread](https://ubuntuforums.org/showthread.php?t=2494273&page=2) The reason behind that was Linux kernel `6.5.0-generic` was compiled by `gcc-12` whereas the driver was compiled by `gcc-11` and the incompatibility on the header files between two compiler versions led to the driver not being recognized by the new kernel.

You can check your gcc version by issuing

    $ gcc --version

The Ubuntuforum thread offered two solutions for this particular issue: either to downgrade your gcc version and downgrade linux kernel altogether and reinstall NVIDIA driver or download a patched driver version from a launchpad site which I believe was compiled by gcc-12 and then reinstall it while the official fix from NVIDIA has not been released yet. I opted in the first option although I tested and could verify that the second option also worked. Details here

```sh
$ sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 11 --slave /usr/bin/g++ g++ /usr/bin/g++-11 --slave /usr/bin/gcov gcov /usr/bin/gcov-11
$ sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 12 --slave /usr/bin/g++ g++ /usr/bin/g++-12 --slave /usr/bin/gcov gcov /usr/bin/gcov-12
$ sudo update-alternatives --config gcc
$ sudo apt remove --purge --yes linux-headers-6.5.0-14-generic linux-image-6.5.0-14-generic linux-modules-6.5.0-14-generic linux-modules-extra-6.5.0-14-generic
$ sudo apt install --reinstall --yes linux-headers-6.2.0-39-generic linux-image-6.2.0-39-generic linux-modules-6.2.0-39-generic linux-modules-extra-6.2.0-39-generic
$ sudo apt-mark hold linux-headers-6.2.0-39-generic linux-image-6.2.0-39-generic linux-modules-6.2.0-39-generic linux-modules-extra-6.2.0-39-generic
# When they come up with a fix to compile NVidia Legacy drivers with Linux kernel 6.5 series kernels, then you can re-add that kernel series by unpinning the kernels via
# sudo apt-mark unhold linux-headers-6.2.0-39-generic linux-image-6.2.0-39-generic linux-modules-6.2.0-39-generic linux-modules-extra-6.2.0-39-generic
# sudo apt update && sudo apt upgrade
# Reboot into 6.2.0-39-generic kernel from GRUB options and reinstall nvidia-driver-390
```

One question crossed my mind which as to how we could verify that the graphic driver had been properly installed.

Firstly one can check if your graphic card has been connected to PCI port. This command should give you similar result independent of the driver being properly installed

    $ lspci | grep -i nvidia
    
```console
01:00.0 VGA compatible controller: NVIDIA Corporation GF119 [GeForce 605] (rev a1)
01:00.1 Audio device: NVIDIA Corporation GF119 HDMI Audio Controller (rev a1)
```
One may as well test the graphic driver using NVIDIA util like

    $ nvidia-smi

```console
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 390.157                Driver Version: 390.157                   |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce 605         Off  | 00000000:01:00.0 N/A |                  N/A |
| 47%   64C    P0    N/A /  N/A |    472MiB /   961MiB |     N/A      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0                    Not Supported                                       |
+-----------------------------------------------------------------------------+
```
### Fix black screen problem after reboot
Actually things didn't work out flawlessly for me, I stumbled across different Ubuntuforums and ended up installing NVIDIA driver on `6.5.0-generic` kernel and then mistakenly remove `xserver-xorg` which is a very BIG issue you might be yelling at me. But I was stupid and I commited the crime anyway. If you run into the problem that the desktop doesn't load and leaves you staring at the black screen, don't worry 'cause I have a fix for you.
Reboot into recovery mode, select something like `Linux kernel 6.5.0-generic (recovery mode)` from GRUB options. Or if you get stuck at the middle of the installation you can hold `Alt + F[1-9]` key. In my case I have `FUJITSU ESPRIMO P910` model desktop at this time of writing it was F2 key which brings you to recovery mode. There you still have full access to file system and services. You're gonna start with reinstalling `xserver-xorg` and `ubuntu-gnome-desktop`

    $ apt-cache search ubuntu-gnome-desktop
    $ sudo apt install xserver-xorg ubuntu-gnome-desktop
    $ sudo reboot

I recall that I even went too far by running
```bash
$ sudo telinit 3
```
which I thought that might add to the problem that the GUI disappeared later. But as you find it out later in the official NVIDIA installation guide it was actually a good practice during installation. Basically it sets the runlevel to 3 which tells the system to disable GUI and that prevents the system from looping the unfinished GUI

### Other ways to verify correct NVIDIA driver and OpenGL installation
If you'd rather write a rudimentary opengl program then here you go (Again Microsoft Bing Copilot did a great job except that it forgot to tell me to link all the necessary libs and I had to iteratively ask for the missing ones 😄)
```C++
#include <GL/glew.h>
#include <GLFW/glfw3.h>

int main() {
    // Initialize the library
    if (!glfwInit())
        return -1;

    // Create a windowed mode window and its OpenGL context
    GLFWwindow* window;
    window = glfwCreateWindow(640, 480, "Hello World", NULL, NULL);
    if (!window) {
        glfwTerminate();
        return -1;
    }

    // Make the window's context current
    glfwMakeContextCurrent(window);

    // Initialize GLEW
    if (glewInit() != GLEW_OK)
        return -1;

    // Loop until the user closes the window
    while (!glfwWindowShouldClose(window)) {
        // Render here
        glClear(GL_COLOR_BUFFER_BIT);

        // Swap front and back buffers
        glfwSwapBuffers(window);

        // Poll for and process events
        glfwPollEvents();
    }

    glfwTerminate();
    return 0;
}
```

After saving the code as `GLTest.cpp`, compile it via
```bash
$ g++ GLTest.cpp -o GLTest -lglfw -lGLEW -lGL
$ ./GLTest
```
At the end you should see a black screen which confirms that OpenGL+NVIDIA Driver stack is working.

Well it's all good and gives you a pleasure of writing OpenGL code, does it? If you are an admin and prefer looking up the installation guide you may consult https://us.download.nvidia.com/XFree86/Linux-x86_64/390.157/README/index.html

This is indeed a very very good resource which you can dig in. To be honest I learned quite a bunch of basic Linux from it.

If `lspci` and `nvidia-smi` both give you normal result we can still verify your OpenGL/NVIDIA stack by means of `glxinfo`
```bash
$ sudo glxinfo
```
If it gives you the following you're good to go 😉
```console
name of display: :1
display: :1  screen: 0
direct rendering: Yes
server glx vendor string: NVIDIA Corporation
server glx version string: 1.4
server glx extensions:
    GLX_ARB_context_flush_control, GLX_ARB_create_context, 
    GLX_ARB_create_context_no_error, GLX_ARB_create_context_profile, 
    GLX_ARB_create_context_robustness, GLX_ARB_fbconfig_float, 
    GLX_ARB_multisample, GLX_EXT_buffer_age, 
    GLX_EXT_create_context_es2_profile, GLX_EXT_create_context_es_profile, 
    GLX_EXT_framebuffer_sRGB, GLX_EXT_import_context, GLX_EXT_libglvnd, 
    GLX_EXT_stereo_tree, GLX_EXT_swap_control, GLX_EXT_swap_control_tear, 
    GLX_EXT_texture_from_pixmap, GLX_EXT_visual_info, GLX_EXT_visual_rating, 
    GLX_NV_copy_image, GLX_NV_delay_before_swap, GLX_NV_float_buffer, 
    GLX_NV_multisample_coverage, GLX_NV_robustness_video_memory_purge, 
    GLX_SGIX_fbconfig, GLX_SGIX_pbuffer, GLX_SGI_swap_control, 
    GLX_SGI_video_sync
client glx vendor string: NVIDIA Corporation
```
On the contrary, if you get some errornous result like 
```console
name of display: :0
Error: couldn't find RGB GLX visual or fbconfig
```
or indirectly from compile log `freeglut (./nbody): OpenGL GLX extension not supported by display ':1'` something is still off. If you run into trouble and does not know what's happening, you can find logs for most of the cases in `/var/log/`

Inspecting Xorg.0.log `grep -i nvidia /var/log/Xorg.0.log` as suggested from https://discussion.fedoraproject.org/t/error-couldnt-find-rgb-glx-visual-or-fbconfig/57646/8 showed me that the nvidia driver cannot be loaded. 
```console
[    28.204] (II) LoadModule: "glx"
[    28.204] (II) Loading /usr/lib/x86_64-linux-gnu/xorg/extra-modules/libglx.so
[    28.279] (EE) Failed to load /usr/lib/x86_64-linux-gnu/xorg/extra-modules  /libglx.so: libnvidia-tls.so.390.157: cannot open shared object file: No such file or directory
[    28.279] (II) UnloadModule: "glx"
[    28.279] (II) Unloading glx
[    28.279] (EE) Failed to load module "glx" (loader failed, 7)
```
More specifically freeglut cannot dynamically link to `libnvidia-tls`, `libnvidia-glcore` and `libnvidia-ml`
```sh
$ ldd /usr/lib/x86_64-linux-gnu/nvidia/xorg/libglx.so
...
libnvidia-tls.so.390.157 => not found
libnvidia-glcore.so.390.157 => not found
...
```
Quick explanation from Stackoverflow: 
> GLX is the X11 protocol extension used to setup OpenGL contexts on X11 drawables. However this is an extension provided by the device driver. You are using a NVidia card. My guess is, that this is a vanilla installation of a system that doesn't automatically install the proprietary nvidia drivers and neither configures the open nouveau drivers.
>
> So the X11 server probably uses either the nv or the fbdev or the vesa driver; none of those support OpenGL/GLX.
>
> Solution: Install and configure the proper driver. Either nouveau or the drivers you can download from http://www.nvidia.com/object/unix.html and install that

## Installing NVIDIA-CUDA-Toolkit

My initial plan was to install Cuda Toolkit in order to run Tensorflow code with GPU accelarated on my Ubuntu machine which I later realized that Tensorflow requires Compute Capability 3.0 :sick. In simple words, CUDA is an abstract layer for developer to write General Purpose GPU code on your graphic card as a hardware. By means of CUDA SDK you can program your Processing Units, make them parallel and able to communicate with CPU without the need to involve with low-level GL operations. Of course CUDA requires that NVIDIA or other provider driver has to be installed beforehand.

One difficulty is that you must determine the right CUDA version for your driver (my old and weak buddy). It is recommended to check the documentation carefully before installation or you'll end up going through vicious circle of installation/removal of CUDA Toolkit. You may need to deep dive into this list

    https://docs.nvidia.com/deploy/cuda-compatibility/index.html#binary-compatibility__table-toolkit-driver (Compatibility concept)
    https://docs.nvidia.com/cuda/archive/11.0/cuda-toolkit-release-notes/index.html#title-new-features (Release Note)
    https://developer.nvidia.com/cuda-gpus (Compute Capability Table)
    https://developer.nvidia.com/cuda-toolkit-archive (Older Toolkit download page)

Now I'm about to note down my installation steps for future reference, it's a reward for your hardwork to keep your old man up and running 😃 
```bash
#if you're installing cuda toolkit 9
$ wget https://developer.download.nvidia.com/compute/cuda/9.0/secure/Prod/local_installers/cuda-repo-ubuntu1704-9-0-local_9.0.176-1_amd64.deb
$ sudo dpkg -i ./cuda-repo-ubuntu1704-9-0-local_9.0.176-1_amd64.deb # if you're installing cuda toolkit 9
$ sudo apt update
$ sudo apt install cuda-toolkit-9-0 
#if you're installing cuda toolkit 5
$ wget https://developer.download.nvidia.com/compute/cuda/5_0/rel-update-1/installers/cuda_5.0.35_linux_64_ubuntu11.10-1.run
$ sudo ./cuda_5.0.35_linux_64_ubuntu11.10-1.run
# set environment variables
$ export PATH=/usr/local/cuda/bin:$PATH
$ export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
$ nvcc --version # it should print out the right version on success
```
Now that you have your CUDA Toolkit installed, the next thing you'll want to do is to navigate to sample folder and then compile it and run the sample application. With CUDA version 9.0 you can find its documentation online at https://docs.nvidia.com/cuda/archive/9.0/cuda-samples/index.html#simple. With version 5.0 the documentation can be found in sample folder. By the time of this writing there was already a github repository for CUDA samples which can be found at https://github.com/NVIDIA/cuda-samples.git. Unfortunately it doesn't support my old buddies 😦 and I have to resort to using sample code shipped with CUDA package. In the following subsection I mean `$CUDA_9_INSTALL_DIR=/usr/local/cuda-9.0` and `$CUDA_5_INSTALL_DIR=/usr/local/cuda-5.0` by convention. Because cuda-5.0 installation kit installs samples in a separate directory as `CUDA_5_INSTALL_DIR`, I use `CUDA_5_SAMPLE_DIR` to denote the sample directory for cuda-5.0 

The very basic application you will want to build is the deviceQuery or deviceQueryDrv which is located at `$CUDA_9_INSTALL_DIR/samples/1_Utilities/deviceQuery[Drv]` with CUDA Toolkit 9.0 and `$CUDA_5_SAMPLE_DIR/1_Utilities/deviceQuery[Drv]` with CUDA Toolkit 5.0 respectively. Now cd into the source directory, create objdir directory (it's always a good practice to create this dir to hold our built artifacts) then cd to it. After running make (cmake) you might run into the following error/warning message
```console
MapSMtoCores for SM 2.1 is undefined. Default to use 128 Cores/SM
# or
CUDA error at bodysystemcuda_impl.h:302 code=13(cudaErrorInvalidSymbol) "setSofteningSquared(softeningSq)"
```
As I mentioned in previous section, you should carefully check your CUDA Compute Capabilities to avoid unnecessary incompatibility problems and now this error message rings a bell that I have too new CUDA version which no longer supports my old graphic driver. My Fermi graphic card has SM 2.1 architecture and it cannot be found by CUDA 9.0 and ironically it defaults to Hopper with 128 Cores which is the latest at this time being. I indeed overlooked the minimum requirement for CUDA-9.0 and installed it despite the requirement. Another tale tell sign to be sure your graphic driver (capability) is support is to look into the Makefile or CMakeLists.txt and search for supported architectures. For example:

in `$CUDA_5_SAMPLE_DIR/1_Utilities/deviceQueryDrv/Makefile`
```make
# CUDA code generation flags
GENCODE_SM10    := -gencode arch=compute_10,code=sm_10
GENCODE_SM20    := -gencode arch=compute_20,code=sm_20
GENCODE_SM30    := -gencode arch=compute_30,code=sm_30 -gencode arch=compute_35,code=sm_35
GENCODE_FLAGS   := $(GENCODE_SM10) $(GENCODE_SM20) $(GENCODE_SM30)
```
in `$CUDA_5_SAMPLE_DIR/5_Simulations/nbody/CMakeLists.txt` 
```cmake
set(GENCODE -gencode=arch=compute_30,code=sm_30 -gencode=arch=compute_30,code=compute_30)
set(GENCODE ${GENCODE} -gencode=arch=compute_20,code=sm_20 -gencode=arch=compute_20,code=compute_20)
```
Note the difference between archtype and code, they don't always map to the same figure. In my case the variables should be passed as `arch=compute_20,code=sm_21`

in `$CUDA_9_INSTALL_DIR/samples/1_Utilities/deviceQuery/Makefile`
```make
# Gencode arguments
SMS ?= 30 35 37 50 52 60 70
```
In any case, you should read the Release Notes to get informed of the new features after each CUDA Toolkit comes around. Back to my problem, as in [insert link here] my graphic card maps to archetype 2.0 and architecture SM2.1 which is rather old, that explains why compilation failed everytime with CUDA 9.0

Once you're done with that basic application you can go ahead building the n-body sample application. One problem I encountered was that one particular CUDA header file purposely refuses to accept higher version of gcc. In particular, after installing CUDA Toolkit 9.0 (or 5.0) and compiled the n-body source code an error was thrown due to this assertion in `$CUDA_9_INSTALL_DIR/include/crt/host_config.h`

```C++
#if __GNUC__ > 6

#error -- unsupported GNU version! gcc versions later than 6 are not supported!

#endif /* __GNUC__ > 6 */
```

and in `CUDA_5_INSTALL_DIR/include/host_config.h`

```C++
#if defined(__GNUC__)

#if __GNUC__ > 4 || (__GNUC__ == 4 && __GNUC_MINOR__ > 6)

#error -- unsupported GNU version! gcc 4.7 and up are not supported!

#endif /* __GNUC__> 4 || (__GNUC__ == 4 && __GNUC_MINOR__ > 6) */

#endif /* __GNUC__ */
```

## Installing older versions of gcc
I decided to make this section short and to the point. In general you have two options: build `gcc` from source code (I'm bad at this although I followed instructions at https://gcc.gnu.org/wiki/InstallingGCC all along) or from downloaded package which is done after finding an appropriate ppa or launchpad repository which covers your old `gcc`. Well the old guys oftentimes team up together 🤷 I resorted to build it the easier way because first of all I'm not confident with figuring out what could go wrong if `make` gets stuck at the middle of the build and I'm pretty sure you won't want to wait for an hour compiling `gcc` from source unless you have a good reason to do so.
```sh
# The following is to install gcc-6.3
$ wget https://drive.google.com/drive/folders/1xVEATaYAwqvseBzYxKDzJoZ4-Hc_XOJm/gcc63-c++-6.3.0-1_amd64.deb
$ sudo dpkg -i gcc63-c++-6.3.0-1_amd64.deb
$ sudo apt update
$ sudo apt install gcc63

# Not necessary, only if you want to add repository
$ sudo vi /etc/apt/sources.list
# add one entry into the list which contains URL and distribution of the repo
$ sudo apt update

# The following is to install gcc-4.0
$ wget https://mirrors.edge.kernel.org/ubuntu/pool/universe/g/gcc-4.6/libstdc++6-4.6-dev_4.6.4-6ubuntu2_amd64.deb
$ wget https://mirrors.edge.kernel.org/ubuntu/pool/universe/g/gcc-4.6/gcc-4.6_4.6.4-6ubuntu2_amd64.deb
$ wget https://mirrors.edge.kernel.org/ubuntu/pool/universe/g/gcc-4.6/g++-4.6_4.6.4-6ubuntu2_amd64.deb
$ wget https://mirrors.edge.kernel.org/ubuntu/pool/universe/g/gcc-4.6/gcc-4.6-base_4.6.4-6ubuntu2_amd64.deb
$ wget https://mirrors.edge.kernel.org/ubuntu/pool/universe/g/gcc-4.6/cpp-4.6_4.6.4-6ubuntu2_amd64.deb
$ wget http://launchpadlibrarian.net/336269522/libmpfr4_3.1.6-1_amd64.deb
$ sudo dpkg -i libmpfr4_3.1.6-1_amd64.deb
$ sudo apt update
$ sudo apt install libmpfr4
$ sudo apt install ./gcc-4.6_4.6.4-6ubuntu2_amd64.deb ./gcc-4.6-base_4.6.4-6ubuntu2_amd64.deb ./libstdc++6-4.6-dev_4.6.4-6ubuntu2_amd64.deb ./cpp-4.6_4.6.4-6ubuntu2_amd64.deb   ./g++-4.6_4.6.4-6ubuntu2_amd64.deb
$ gcc-4.6 --version

# Configure default gcc when you have multiple gcc versions installed
$ sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc63 90 --slave /usr/bin/g++ g++ /usr/bin/g++63
$ sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 80 --slave /usr/bin/g++ g++ /usr/bin/g++-11 --slave /usr/bin/gcov gcov /usr/bin/gcov-11
$ sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 70 --slave /usr/bin/g++ g++ /usr/bin/g++-12 --slave /usr/bin/gcov gcov /usr/bin/gcov-12
$ sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.6 110 --slave /usr/bin/g++ g++ /usr/bin/g++-4.6
```
### Patching GLIBC package (no not really, but I can't think of other name for this section)
Another problem I encountered was that since the release of glibc-xx it seemed to tamper with part of the system which supports large complex number, as we name it `float128`

To check glibc version, do this following command

    $ ldd --version

```console    
ldd (Ubuntu GLIBC 2.35-0ubuntu3.7) 2.35
Copyright © 2022 Free Software Foundation, Inc.
Dies ist freie Software; in den Quellen befinden sich die Lizenzbedingungen.
Es gibt KEINERLEI Garantie; nicht einmal für die TAUGLICHKEIT oder
VERWENDBARKEIT FÜR EINEN ANGEGEBENEN ZWECK.
Implementiert von Roland McGrath und Ulrich Drepper.
```
Another command give you a slightly different result
```console
GNU C Library (Ubuntu GLIBC 2.35-0ubuntu3.7) stable release version 2.35.
    Copyright (C) 2022 Free Software Foundation, Inc.
    This is free software; see the source for copying conditions.
    There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
    PARTICULAR PURPOSE.
    Compiled by GNU CC version 11.4.0.
    libc ABIs: UNIQUE IFUNC ABSOLUTE
    For bug reporting instructions, please see:
    <https://bugs.launchpad.net/ubuntu/+source/glibc/+bugs>.
```
A short background about ldd: It is a standard dynamic linker which can inspect the program's dynamic dependencies and load the objects that satisfy dependencies.

To fix this, firstly cd into `/usr/include/x86_64-linux-gnu/bits` (In other Linux system it may be different than this, for example `/usr/include/bits/floatn.h`), create a new file named `floatn.h.patch` and paste the following content into it. Don't forget to do it in `sudo` mode 😉

```patch
--- a/usr/include/bits/floatn.h 2017-09-13 16:00:37.079654880 +0200
+++ b/usr/include/bits/floatn.h 2017-09-13 16:00:00.999104815 +0200
@@ -31,7 +31,8 @@
    support, for x86_64 and x86.  */
 #if (defined __x86_64__                                                        \
      ? __GNUC_PREREQ (4, 3)                                            \
-     : (defined __GNU__ ? __GNUC_PREREQ (4, 5) : __GNUC_PREREQ (4, 4)))
+     : (defined __GNU__ ? __GNUC_PREREQ (4, 5) : __GNUC_PREREQ (4, 4))) \
+  &&  !defined(__CUDACC__)
 # define __HAVE_FLOAT128 1
 #else
 # define __HAVE_FLOAT128 0
```
After that apply the patch

    $ sudo patch floatn.h floatn.h.patch
    
If it logs out this, you're one step further 😙
```console
patching file floatn.h
Hunk #1 succeeded at 28 (offset -3 lines).
```
The cleaner way would be to download glibc-source, create a patch from that source and then apply the patch on your system. But I was too lazy to try it out and was ready to take risk of breaking other packages ☹️

To fix 'dso-missing' linking error we can add customized option to gcc: 
```sh
export LDFLAGS="-Wl,--copy-dt-needed-entries"
```
See https://linuxpip.org/how-to-fix-dso-missing-from-command-line?utm_content=cmp-true 

In CMakeLists.txt we add the following line in CMakeLists to set option for target link
```cmake
target_link_options(nbody PRIVATE "LINKER:--copy-dt-needed-entries")
```
And adding two more OpenGl libraries to the original CMakeLists.txt file seems to fix the last hurdle!
```cmake
if (WIN32)
  if( CMAKE_SIZEOF_VOID_P EQUAL 8 )
    set( LIB_PATH ${CUDA_SDK_ROOT_DIR}/common/lib/x64/ )
    set( GLEW_NAME glew64 )
  else( CMAKE_SIZEOF_VOID_P EQUAL 8 )
    set( LIB_PATH ${CUDA_SDK_ROOT_DIR}/common/lib/win32/ )
    set( GLEW_NAME glew32 )
  endif( CMAKE_SIZEOF_VOID_P EQUAL 8 )
else (WIN32)
  if( CMAKE_SIZEOF_VOID_P EQUAL 8 )
    set( LIB_PATH ${CUDA_SDK_ROOT_DIR}/common/lib/linux/x86_64/ )
    set( GLEW_NAME "libGLEW.so" ) # this was hardcoded 
    set(GLU_NAME "libGLU.so") # this was also hardcoded
  else( CMAKE_SIZEOF_VOID_P EQUAL 8 )
    set( LIB_PATH ${CUDA_SDK_ROOT_DIR}/common/lib/linux/i686/ )
    set( GLEW_NAME GLEW )  
  endif( CMAKE_SIZEOF_VOID_P EQUAL 8 )
endif (WIN32)

if (WIN32)
  FIND_LIBRARY(GLEW_LIBRARY NAMES ${GLEW_NAME} PATHS ${LIB_PATH})
else (WIN32)
  message("GLEW_NAME=" ${GLEW_NAME})
  FIND_LIBRARY(GLEW_LIBRARY NAMES ${GLEW_NAME} PATHS ${LIB_PATH})
  FIND_LIBRARY(GLU_LIBRARY NAMES ${GLU_NAME} PATHS ${LIB_PATH})
endif (WIN32)
```
Rerun cmake and make and finally

[Bildschirmaufzeichnung vom 22.04.2024, 12:22:27.webm](https://github.com/chut89/NVIDIA-Driver390-Setup/assets/25095256/30633238-a47b-4bda-a17f-9143c8089945)

Voila! We made it! Although not 100% as my original plan because Tensorflow supports SM 3.0 at the minimum but still we've been through a lot and learned a lot from this small experiment!

The whitepaper was uploaded to this repository in case you want to read it.
