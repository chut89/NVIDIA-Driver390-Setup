# NVIDIA-Driver390-Setup #
This documentation walks you through the installation of NVIDIA driver version 390.157 and CUDA-Toolkit-5.0 on Ubuntu 22.04 (kernel 6.2.0-39-generic). At the end of this guide you should be able to render the result of nbody sample from CUDA-Toolkit.

Note: During this documentation, feel free to `man page` every command you're not sure of

## Installing Nvidia driver ##
It all began when I upgraded my Ubuntu kernel version from `6.2.0-39-generic` to `6.5.0-generic` and my NVIDIA graphic card stopped working properly. 

You can issue the kernel version via
    
    $ uname -r

```
6.2.0-39-generic
```
If you just want a short name of your system, normally an OS name you can do it via
    
    $ uname -m

```
Linux
```
A full system description can be issued via

    $ uname -a

On Ubuntu when you click Power button\Settings\Info your Graphic Setting should give you detail like

    GeForce 605/PCIe/SSE2

and not

    llvmpipe

When the latter renderer is used which normally means the NVIDIA is not working properly it gives you very poor graphic quality which discourages you to use your machine a lot :-) Asking Microsoft Bing Copilot led me to [THIS IS BROKEN](https://github.com/batocera-linux/batocera.linux/issues/9569) The reason behind that was Linux kernel `6.5.0-generic` was compiled by `gcc-12` whereas the driver was compiled by `gcc-11` and the incompatibility on the header files between two compiler versions led to the driver not being recognized by the new kernel.

You can check your gcc version by issuing

    $ gcc --version

The Ubuntuforum thread offered two solutions for this particular issue: either to downgrade your gcc version and downgrade linux kernel altogether and reinstall NVidia driver or download a patched driver version from a launchpad site which I believe was compiled by gcc-12 and then reinstall it while the official fix from NVidia has not been released yet. I opted in the first option although I tested and could verify that the second option also worked.

One question crossed my mind which as to how we could verify that the graphic driver had been properly installed.

Firstly one can check if your graphic card has been connected to PCI port. This command should give you similar result independent of the driver being properly installed

    $ lspci | grep -i nvidia
    
```
01:00.0 VGA compatible controller: NVIDIA Corporation GF119 [GeForce 605] (rev a1)
01:00.1 Audio device: NVIDIA Corporation GF119 HDMI Audio Controller (rev a1)
```
One may as well test the graphic driver using NVidia util like

    $ nvidia-smi

```
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
Actually things didn't work out flawlessly for me, I stumbled across different Ubuntuforums and ended up installing NVidia driver on `6.0.5-generic` kernel and then mistakenly remove `xserver-xorg` which is a very BIG issue you might be yelling at me. But I was stupid and I commited the crime anyway. If you run into the problem that the desktop doesn't load and leaves you staring at the black screen, don't worry 'cause I have a fix for you.
Reboot into recovery mode, select something like `Linux kernel 6.0.5-recovery mode` on GRUB selection. Or if you get stuck at the middle of the installation you can hold `Alt + F[1-9]` key. In my case it was F2 key which brings you to recovery mode. There you still have full access to file system and services. You're gonna start with reinstalling `xserver-xorg` and `ubuntu-gnome-desktop`

    $ apt-cache search ubuntu-gnome-desktop
    $ sudo apt install xserver-xorg ubuntu-gnome-desktop
    $ sudo reboot

I recall that I even went too far by running

    sudo telinit 3

which I thought that might add to the problem that the GUI disappeared later. But as you find it out later in the official NVidia installation guide it was actually a good practice during installation. Basically it sets the system level to 3 which tells the system to disable GUI and that prevents the system from looping the unfinished GUI

### Other ways to verify correct Nvidia driver and OpenGL installation
If you'd rather write a rudimentary opengl program then here you go (Again Microsoft Bing Copilot did a great job except that it forgot to tell me to link all the necessary libs and I had to iteratively ask for the missing ones :D)
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

    g++ GLTest.cpp -o GLTest -lglfw -lGLEW -lGL
    ./GLTest

At the end you should see a black screen which confirms that OpenGL+NVIdia Driver stack is working.

Well it's all good and gives you a pleasure of writing OpenGL code, does it? If you are an admin and prefer looking up the installation guide then you should consult https://us.download.nvidia.com/XFree86/Linux-x86_64/390.157/README/index.html

This is indeed a very very good resource which you can dig in. To be honest I learned quite a bunch of basic Linux from it. A good example is that I learnt how to use `glxinfo` from that

    sudo glxinfo

If it gives you the following you're good to go 😉

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

If you run into trouble and does not know what's happening, you can find logs for most of the cases in `/var/log/`

## Installing NVidia-CUDA-Toolkit
My initial plan was to install Cuda Toolkit in order to run Tensorflow code with GPU accelarated on my Ubuntu machine which I later realized that Tensorflow requires Compute Capability 3.0 :sick. In simple words, CUDA is an abstract layer for developer to write General Purpose GPU code on your graphic card as a hardware. By means of CUDA SDK you can program your Processing Units, make them parallel and able to communicate with CPU without the need to involve with low-level GL operations. Of course CUDA requires that NVidia or other provider driver has to be installed beforehand.

One difficulty is that you must determine the right CUDA version for your driver (my old and weak buddy). It is recommended to check the documentation carefully before installation or you'll end up going through vicious circle of installation/removal of CUDA Toolkit. You may need to deep dive into this list

    https://docs.nvidia.com/deploy/cuda-compatibility/index.html#binary-compatibility__table-toolkit-driver (Compatibility concept)
    https://docs.nvidia.com/cuda/archive/11.0/cuda-toolkit-release-notes/index.html#title-new-features (Release Note)
    https://developer.nvidia.com/cuda-toolkit-archive (Older Toolkit download page)

Now I'm about to note down my installation steps for future reference, it's a reward for your hardwork to keep your old man up and running 😃

    $ wget https://developer.download.nvidia.com/compute/cuda/9.0/secure/Prod/local_installers/cuda-repo-ubuntu1704-9-0-local_9.0.176-1_amd64.deb
    $ sudo dpkg -i ./cuda-repo-ubuntu1704-9-0-local_9.0.176-1_amd64.deb # if you're installing cuda toolkit 9

    #if you're installing cuda toolkit 9
    $ sudo apt update
    $ sudo apt install cuda-toolkit-9-0 

    #if you're installing cuda toolkit 5
    $ sudo ./cuda_5.0.35_linux_64_ubuntu11.10-1.run
    $ nvcc --version # it should print out the right version on success

## Installing older versions of gcc
I decided to make this section short and to the point. In general you have two options: build `gcc` from source code (I'm bad at this although I followed instructions at https://gcc.gnu.org/wiki/InstallingGCC all along) or from downloaded package which is done after finding an appropriate ppa or launchpad repository which covers your old `gcc`. Well the old guys oftentimes team up together 🤷 I resorted to build it the easier way

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

This page should help you install gcc-xx well 
```diff
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

    sudo patch floatn.h floatn.h.patch
    
If it logs out this, you're one step further 😙

    patching file floatn.h
    Hunk #1 succeeded at 28 (offset -3 lines).

To fix linking error: 

    export LDFLAGS="-Wl,--copy-dt-needed-entries"

See https://linuxpip.org/how-to-fix-dso-missing-from-command-line?utm_content=cmp-true 

Add the following line in CMakeLists to set option for target link

    CMake `target_link_options(nbody PRIVATE "LINKER:--copy-dt-needed-entries")`
