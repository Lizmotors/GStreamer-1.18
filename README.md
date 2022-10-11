# GStreamer-1.18
Install GStreamer 1.18 on Raspberry Pi 4.

ntroduction.
GStreamer is a pipeline-based multimedia framework that links various media processes to a complex workflow. For example, with a single line of code, it can retrieve images from a camera, convert them to Mpeg, and send them as UDP packets over Ethernet to another computer. Obviously, GStreamer is complex software used by more advanced programmers.

One of the main reasons for using GStreamer is the lack of latency. The OpenCV video capture module uses large video buffers, holding the frames. For example, if your camera has a frame rate of 30 FPS and your image processing algorithm can handle a maximum of 20 FPS, the synchronisation is lost very quickly due to the stacking in the video buffer. The absence of buffer flushing makes things even worse.
In this situation, GStreamer comes to the rescue. With the buffering of just one frame, you will always get an actual frame as output.

Recently the Raspberry Pi has released the Bullseye operating system. One of the changes compared to older Buster version is the absence of the Userland video engine. It leaves GStreamer as one of the default methods for capturing live video. At the same time, Bullseye uses now GStreamer 1.18.4. The Bullseye section below gives you the detailed information.

BullseyeBuster

The guide covers the following topics.
Version 1.18.4. We start with a survey of the current version 1.18.4 of Bullseye.
Version 1.14.4. The default Buster GStreamer version 1.14.4 and the installation of rpicamsrc on 32-bit systems.
Version 1.18.4. The second part covers installing GStreamer 1.18.4 on your Raspberry Pi Buster.
Streaming examples. In the last part, a lot of streaming examples, including streaming to YouTube are explored.
Bullseye Version 1.18.4.

When you scan your system for GStreamer, you will find several packages of version 1.18.4 already installed. They are essential for your Raspberry desktop.

LibGStreamer1_18_4

There are a few additional plugins you must install before you can stream live video. For those familiar with the previous 1.14.4 version of Buster, it's almost the same list of libraries. Please, follow the commands below.
# install a missing dependency
$ sudo apt-get install libx264-dev libjpeg-dev
# install the remaining plugins
$ sudo apt-get install libgstreamer1.0-dev \
     libgstreamer-plugins-base1.0-dev \
     libgstreamer-plugins-bad1.0-dev \
     gstreamer1.0-plugins-ugly \
     gstreamer1.0-tools \
     gstreamer1.0-gl \
     gstreamer1.0-gtk3
# if you have Qt5 install this plugin
$ sudo apt-get install gstreamer1.0-qt5
# install if you want to work with audio
$ sudo apt-get install gstreamer1.0-pulseaudio


Streaming.
With all GStreamer modules installed let's test the installation with $ gst-launch-1.0 videotestsrc ! videoconvert ! autovideosink .

BullseyeTestSrc

The major difference between Buster and Bullseye is the camera source. In Buster, you could call v4l2src device=/dev/video0 or rpicamsrc. In Bullseye, the only working camera source now is libcamerasrc. An example $ gst-launch-1.0 libcamerasrc ! video/x-raw, width=640, height=480, framerate=30/1 ! videoconvert ! videoscale ! clockoverlay time-format="%D %H:%M:%S" ! autovideosink

BullseyeGStreamer

Keep this in mind when composing your pipelins; use libcamerasrc in Bullseye.
LCCV.
If you only want a working camera in OpenCV, consider LCCV. A few lines of C++ code integrating low-level libcamera routines with OpenCV. It requires fewer resources and also consumes less CPU time. You can find LCCV on our GitHub.
Raspberry Pi OS (Legacy).
Bullseye has a lot of changes under the hood. Suddenly, many users are facing missing libraries causing their software to stop working.
For example, consider the missing v4l2-ctl camera interface. That's why the Raspberry Foundation has restored some of the 'old' features with a legacy version of the Raspberry Pi Buster OS. The release is frozen and not supported. You can install this plugin by using the raspi-config tool. Find more information here.
For the GStreamer, this legacy version means the replacement of libcamerasrc by the well-known v4l2src device=/dev/video0. The raspicamsrc is still deprecated in the old version. If you plan on using the older Raspberry Pi OS, all the examples in the last section will work with the v4l2src device=/dev/video0 source. Ignore the Bullseye hint.
Buster Version 1.14.4.

When you scan your system for GStreamer, you will find several packages already installed. They are essential for your Raspberry desktop.

LibGStreamer1_14_4_2

There are a few additional plugins you can install. These are especially useful if you want to start streaming, one of GStreamer's popular applications.
Please follow the commands below.


# install a missing dependency
$ sudo apt-get install libx264-dev libjpeg-dev
# install the remaining plugins
$ sudo apt-get install libgstreamer1.0-dev \
     libgstreamer-plugins-base1.0-dev \
     libgstreamer-plugins-bad1.0-dev \
     gstreamer1.0-plugins-ugly \
     gstreamer1.0-tools
# install some optional plugins
$ sudo apt-get install gstreamer1.0-gl gstreamer1.0-gtk3
# if you have Qt5 install this plugin
$ sudo apt-get install gstreamer1.0-qt5
# install if you want to work with audio
$ sudo apt-get install gstreamer1.0-pulseaudio
Streaming.
With all GStreamer modules installed let's test the installation with $ gst-launch-1.0 videotestsrc ! videoconvert ! autovideosink .

GStreamer_1_14_4

The Raspicam can be invoked with this rather large pipeline $ gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw, width=1280, height=720, framerate=30/1 ! videoconvert ! videoscale ! clockoverlay time-format="%D %H:%M:%S" ! video/x-raw, width=640, height=360 ! autovideosink Remember to enable the Raspicam on forehand in your Raspberry Pi configuration menu.

GStreamer_Raspicam

All pipeline commands are constructed in the same way. First, the source is named, followed by several operations, after which the sink is determined. All parts of the pipeline are separated from each other by exclamation marks. For instance, in the example above, you could remove the part, which prints the date and time on the screen.  
Common sense is required when composing the pipeline. The limited computing power of the Raspberry Pi does not allow for overly complex pipelines.
UDP streaming.
There are many types of streaming possible with GStreamer. UDP and TCP are most used to connect two devices. The name of the streaming refers to the Ethernet protocol used.
Let's start with UDP. We use two Raspberry Pis, both connected to the same home network. However, it could just as easily be an RPi and a laptop on the other side of the world. You need to know the address of the receiving Raspberry Pi on forehand. Follow the commands below.


Raspberry Pi 32 or 64-bit OS
# get the IP address of the recieving RPi first
$ hostname -I
# start the sender, the one with the Raspicam
$ gst-launch-1.0 -v v4l2src device=/dev/video0 num-buffers=-1 ! video/x-raw, width=640, height=480, framerate=30/1 ! videoconvert ! jpegenc ! rtpjpegpay ! udpsink host=192.168.178.84 port=5200
# start the reciever, the one with IP 192.168.178.84
$ gst-launch-1.0 -v udpsrc port=5200 ! application/x-rtp, media=video, clock-rate=90000, payload=96 ! rtpjpegdepay ! jpegdec ! videoconvert ! autovideosink
Now you can see claerly the structure of a pipline. The sender has the Raspicam located at /dev/video0 as source and sinks to the IP address of the other Raspberry Pi. The recieving RPi has UDP port 5200 as source and sinks to the screen (autovideosink).

UDPsend

none

TCP streaming.
The other method of streaming is with TCP. The difference with UDP is the latency. UDP is faster.
The commands as listed below. Note the different IP addresses. With TCP streaming, you use the server address, the sender, instead of the receiver, as we saw with the UDP streaming.


Raspberry Pi 32 or 64-bit OS
# get the IP address of the sending RPi first
$ hostname -I
# start the sender, the one with the Raspicam and IP 192.168.178.32
$ gst-launch-1.0 -v v4l2src device=/dev/video0 num-buffers=-1 ! video/x-raw,width=640,height=480, framerate=30/1 ! videoconvert ! jpegenc ! tcpserversink  host=192.168.178.32 port=5000
# start the reciever and connect to the server with IP 192.168.178.32
$ gst-launch-1.0 tcpclientsrc host=192.168.178.32 port=5000 ! jpegdec ! videoconvert ! autovideosink
Both streams, UDP and TCP, start with single frames (video/x-raw). A timestamp is inserted if necessary. Then the image is compressed with jpeg to reduce its size, decreasing the required bandwidth. Once received, the jpeg image will be decompressed and displayed on the screen. You can always change the resolution. Frame sizes of 1280x960 at 30 FPS were no problem here at the office.
RTSP streaming.
If you want to stream RTSP (Real-Time Streaming Protocol), you need a server. GStreamer has its own server available for RTSP. If you don't want to stream RTSP, this additional software isn't necessary. The streaming examples section provides some pipelines and other information about setting up an RTSP stream. For now, just the installation commands.

# install the rtsp server version 1.14.4
$ wget https://gstreamer.freedesktop.org/src/gst-rtsp-server/gst-rtsp-server-1.14.4.tar.xz
$ tar -xf gst-rtsp-server-1.14.4.tar.xz
$ cd gst-rtsp-server-1.14.4
$ ./configure
$ make
$ sudo make install
$ sudo ldconfig

rpicamsrc.
Thanks to the impressive work of Jan Schmidt (thaytan), GStreamer now fully support the Raspicam. With the rpicamsrc source installed, you can now pass almost the same parameters to your pipeline as you would to the well-known raspivid application. For example, flip horizontally hflip=true, flip vertically vflip=true, preview=true, or in case of halogen lighting awb-mode=tungsten.
Nearly all of these commands are executed in the image sensor chip, unloading the CPU/GPU on your Raspberry Pi. More information about the rpicamsrc and its commands can be found on the GStreamer site.
The rpicamsrc source is incorporated in the 1.18.4 version. In the case of default Raspberry Pi version 1.14.4, you have to install the plugin yourself.

👉 Please note, rpicamsrc works only on a 32-bit operating system. Due to missing Userland components, it will not work on a 64-bit OS. Use the default v4l2src device=/dev/video0 source in this situation, as shown in the above examples.


Only Raspberry Pi 32-bit OS
# install rpicamsrc in 1.14.4
$ git clone https://github.com/thaytan/gst-rpicamsrc.git
$ cd gst-rpicamsrc
$ ./autogen.sh
$ make
$ sudo make install
$ sudo ldconfig
RpicamsrcMake

Check with $ gst-inspect-1.0 rpicamsrc.

RpiSrc

The UDP or TCP streaming pipelines are shown below. We have added some raspivid commands to the pipeline in green. And for fun, we've put timestamps in the corners of the videos.
ONLY Raspberry Pi 32-bit OS with rpicamsrc
UDP
# get the IP address of the recieving RPi first
$ hostname -I
# start the sender, the one with the Raspicam
$ gst-launch-1.0 -v rpicamsrc num-buffers=-1 ! video/x-raw, width=640, height=480, framerate=30/1 ! clockoverlay time-format="%D %H:%M:%S" ! videoconvert ! jpegenc ! rtpjpegpay ! udpsink host=192.168.178.84 port=5200
# start the reciever, the one with IP 192.168.178.84
$ gst-launch-1.0 -v udpsrc port=5200 ! application/x-rtp, media=video, clock-rate=90000, payload=96 ! rtpjpegdepay ! jpegdec ! videoconvert ! autovideosink
TCP
# get the IP address of the sending RPi first
$ hostname -I
# start the sender, the one with the Raspicam and IP 192.168.178.32
$ gst-launch-1.0 -v rpicamsrc vflip=true awb-mode=tungsten preview=false num-buffers=-1 ! video/x-raw,width=640,height=480, framerate=30/1 ! timeoverlay time-mode="buffer-time" ! videoconvert ! jpegenc ! tcpserversink  host=192.168.178.32 port=5000
# start the reciever and connect to the server with IP 192.168.178.32
$ gst-launch-1.0 tcpclientsrc host=192.168.178.32 port=5000 ! jpegdec ! videoconvert ! autovideosink


Buster Version 1.18.4.
Version 1.18.4.
This section walks you through the installation of GStreamer 1.18 on a Raspberry Pi 4 with a Buster operating system. With version 1.18, GStreamer fully supports the Raspicam on 32-bits operating systems. Unfortunately not on the 64-bits systems, due to the missing Userland video engine.

Because GStreamer is deeply embedded in the Raspberry Desktop, it is not easy to customize or upgrade. Parts of the old version must remain active, which sometimes causes unexpected reactions. Removing the old version entirety will destroy your desktop. It will fall back to the default Debian LXDE desktop.

LXDE desktop

Completely removing the old version will also remove the necessary dependencies. The newly installed version will therefore lack many features.

👉 By the way, keep in mind that you will have to reinstall OpenCV once GStreamer 1.18 is installed.
Version 1.19.2.
On September 23, 2021, Version 1.19.2 is released. There is not much new to report regarding the Raspberry Pi. Version 1.19.2 is more or less an 'intermediate version' pointed out by the GStreamer community here. The most important fact is the Rust implementation in this version, a language only a few uses on an RPi.You can install GStreamer version 1.19.2 on your Raspberry Pi without any problems. Just change the numbering in the commands from 1.18.4 to 1.19.2, and you're ready. Because some plugins have moved among themselves, you cannot combine versions 18 and 19. Put either the complete version 18 or the complete version 19 on your Raspberry Pi, no mix.
Preparation.
You have to install at least three GStreamer packages, the core gstreamer, the plugins-base and the plugins-good. As mentioned, OpenCV has to be rebuilt also, after installing GStreamer. But first remove, if available, earlier user-installed GStreamer versions.

# remove the old version
$ sudo rm -rf /usr/bin/gst-*
$ sudo rm -rf /usr/include/gstreamer-1.0
# install a few dependencies
$ sudo apt-get install cmake meson flex bison
$ sudo apt-get install libglib2.0-dev libjpeg-dev libx264-dev
$ sudo apt-get install libgtk2.0-dev libcanberra-gtk* libgtk-3-dev
# needed for alsasrc, alsasink
$ sudo apt-get install libasound2-dev

Installation.
Next, install the core GStreamer libraries. Again, you just need to install the GStreamer in Buster. Bullseye already pre-installed 1.18.4.

Core (only Buster)
# download and unpack the lib
$ wget https://gstreamer.freedesktop.org/src/gstreamer/gstreamer-1.18.4.tar.xz
$ sudo tar -xf gstreamer-1.18.4.tar.xz
$ cd gstreamer-1.18.4
# make an installation folder
$ mkdir build && cd build
# run meson (a kind of cmake)
$ meson --prefix=/usr \
        --wrap-mode=nofallback \
        -D buildtype=release \
        -D gst_debug=true \
        -D package-origin=https://gstreamer.freedesktop.org/src/gstreamer/ \
        -D package-name="GStreamer 1.18.4 BLFS" ..


If everything went well, you end up with the screen below.

Meson_Rdy_1

Now build and test GStreamer with the next commands.

# build the software
$ ninja -j4
# test the software (optional)
$ ninja test
# install the libraries
$ sudo ninja install
$ sudo ldconfig

Meson_Test_1
Testing.
It is common for some tests to fail. As long as the ninja build doesn't throw an error, don't worry.
Tests fail mainly for two reasons.
First, the required hardware (GPU) is not available on the Raspberry Pi. While you may think it shouldn't be compiled, sometimes Gstreamer needs parts of these libraries later in the build.
Second, most commonly, it takes too much time on a (simple) Raspberry Pi to run the test. Think of parsing a MPEG stream into the original frames. Now an RPi had a lot of work ahead of it. The test may take a certain amount of time, after which it will fail.
When you later install the Bad or Ugly package, even more tests will fail.
OS check.
Please check your operating system to be sure, before the next step. Run the command uname -a and verify your version with the screen dump below.

Version_32_64
Plugins-base.
Once the core libraries are installed, the next step is to install the two additional plugins. Both follow the same procedure as above, so we'll give the installation without much explanation.
However, keep one thing in mind. GStreamer scans your system for available libraries before it starts building the plugins. Even if a particular library is missing, the plugin will still be created. Only the operation that requires the missing library is not functional.
This is a bit confusing. You have your plugin up and running, but it still gives errors on some operation. You are probably missing a library on which the operation depends. In that case, you need to rebuild the plugin after installing the missing library.
64 bit OS
Plugins Base (only Buster)
# some additional dependencies
$ sudo apt-get install libegl1-mesa-dev libglfw3-dev libgles2-mesa-dev
$ cd ~
# download and unpack the plug-ins base
$ wget https://gstreamer.freedesktop.org/src/gst-plugins-base/gst-plugins-base-1.18.4.tar.xz
$ sudo tar -xf gst-plugins-base-1.18.4.tar.xz
# make an installation folder
$ cd gst-plugins-base-1.18.4
$ mkdir build
$ cd build
# run meson
$ meson --prefix=/usr \
-D gl_winsys=wayland \
-D buildtype=release \
-D package-origin=https://gstreamer.freedesktop.org/src/gstreamer/ ..
$ ninja -j4
# optional
$ ninja test
# install the libraries
$ sudo ninja install
$ sudo ldconfig
32 bit OS


Not all tests will pass on a 64-bit system. This is normal due to unsupported features in the Wayland protocol. The screen dump below shows the result of the 32-bit version.

Meson_Test_1

Plugins-good.
The last step is the installation of the plugins-good package. Here you see the earlier mentioned silent library dependency working. Note the installation of libjpeg-dev. This library is required by the jpegenc and jpegdec operations used in our streaming examples.
Plugins Good
$ cd ~
$ sudo apt-get install libjpeg-dev
# download and unpack the plug-ins good
$ wget https://gstreamer.freedesktop.org/src/gst-plugins-good/gst-plugins-good-1.18.4.tar.xz
$ sudo tar -xf gst-plugins-good-1.18.4.tar.xz
$ cd gst-plugins-good-1.18.4
# make an installation folder
$ mkdir build && cd build
# run meson
$ meson --prefix=/usr       \
       -D buildtype=release \
       -D package-origin=https://gstreamer.freedesktop.org/src/gstreamer/ \
       -D package-name="GStreamer 1.18.4 BLFS" ..
$ ninja -j4
# optional
$ ninja test
# install the libraries
$ sudo ninja install
$ sudo ldconfig


Plugins-bad.
There are two additional packages that you should be aware of. The plugins-bad does not meet the same high quality standard as the other pacakges. They may not have been thoroughly tested, or some documentation is missing or in progress. Commonly used format conversion tools, such as h264parse or matroskamux, can be found in this plugin. Keep in mind that the Raspberry Pi 4 still has modest computing power, so converting high frame rates live from MPEG to Matroska will give disappointing results.
Note the additional installations of librtmp-dev and libvo-aacenc-dev that are required if you want to start streaming to YouTube with RTMP.
Plugins Bad
$ cd ~
# dependencies for RTMP streaming (YouTube)
$ sudo apt install librtmp-dev
$ sudo apt-get install libvo-aacenc-dev
# download and unpack the plug-ins bad
$ wget https://gstreamer.freedesktop.org/src/gst-plugins-bad/gst-plugins-bad-1.18.4.tar.xz
$ sudo tar -xf gst-plugins-bad-1.18.4.tar.xz
$ cd gst-plugins-bad-1.18.4
# make an installation folder
$ mkdir build && cd build
# run meson
$ meson --prefix=/usr       \
       -D buildtype=release \
       -D package-origin=https://gstreamer.freedesktop.org/src/gstreamer/ \
       -D package-name="GStreamer 1.18.4 BLFS" ..
$ ninja -j4
# optional
$ ninja test
# install the libraries
$ sudo ninja install
$ sudo ldconfig


Plugins-ugly.
The other package is the plugins ugly. The code is good but may have problems distributing. There may also be patent issues with the libraries on where the plugin depends. See this GStreamer site for more information.


Plugins Ugly
$ cd ~
# download and unpack the plug-ins ugly
$ wget https://gstreamer.freedesktop.org/src/gst-plugins-ugly/gst-plugins-ugly-1.18.4.tar.xz
$ sudo tar -xf gst-plugins-ugly-1.18.4.tar.xz
$ cd gst-plugins-ugly-1.18.4
# make an installation folder
$ mkdir build && cd build
# run meson
$ meson --prefix=/usr       \
      -D buildtype=release \
      -D package-origin=https://gstreamer.freedesktop.org/src/gstreamer/ \
      -D package-name="GStreamer 1.18.4 BLFS" ..
$ ninja -j4
# optional
$ ninja test
# install the libraries
$ sudo ninja install
$ sudo ldconfig
As mentioned, a needed module may not be present in the current installation. If all four plugins are installed, chances are the underlying library isn't installed on your Raspberry Pi. You need to install the required library first and then rebuild the plugin. As an example, the missing x264enc module, found in the ugly package, is installed below. Note that there is no need to recompile OpenCV in this case.

# test if the module exists (for instance x264enc)
$ gst-inspect-1.0 x264enc
# if not, make sure you have the libraries installed
# stackoverflow is your friend here
$ sudo apt-get install libx264-dev
# check which the GStreamer site which plugin holds the module
# rebuild the module (in this case the ugly)
$ cd gst-plugins-ugly-1.18.4
# remove the previous build
$ rm-rf build
# make a new build folder
$ mkdir build && cd build
$ meson --prefix=/usr       \
      -D buildtype=release \
      -D package-origin=https://gstreamer.freedesktop.org/src/gstreamer/ \
      -D package-name="GStreamer 1.18.4 BLFS" ..
$ ninja -j4
$ sudo ninja install
$ sudo ldconfig

omxh264enc.

You can use the omxh264enc plugin as an alternative to the v4l2h264enc plugin. The installation will only succeed if you have a 32-bit operating system. On a 64-bit system, it will fail due to missing Userland components. Note that the installed omxh264enc only accepts raw and h264 video streams as input.


omxh264enc (only 32 bit OS)
$ cd ~
# download and unpack the plug-in gst-omx
$ wget https://gstreamer.freedesktop.org/src/gst-omx/gst-omx-1.18.4.tar.xz
$ sudo tar -xf gst-omx-1.18.4.tar.xz
$ cd gst-omx-1.18.4
# make an installation folder
$ mkdir build && cd build
# run meson
$ meson --prefix=/usr       \
       -D header_path=/opt/vc/include/IL \
       -D target=rpi \
       -D buildtype=release ..
$ ninja -j4
# optional
$ ninja test
# install the libraries
$ sudo ninja install
$ sudo ldconfig
RTSP server.
If you want to stream RTSP (Real-Time Streaming Protocol), you need the GStreamer RTSP server. The installation is slightly different from version 1.14.4 but in line with the version 1.18.4 plugins.

RTSP Server
$ cd ~
# install the rtsp server version 1.18.4
$ wget https://gstreamer.freedesktop.org/src/gst-rtsp-server/gst-rtsp-server-1.18.4.tar.xz
$ tar -xf gst-rtsp-server-1.18.4.tar.xz
$ cd gst-rtsp-server-1.18.4
# make an installation folder
$ mkdir build && cd build
# run meson
$ meson --prefix=/usr       \
       --wrap-mode=nofallback \
       -D buildtype=release \
       -D package-origin=https://gstreamer.freedesktop.org/src/gstreamer/ \
       -D package-name="GStreamer 1.18.4 BLFS" ..
$ ninja -j4
# install the libraries
$ sudo ninja install
$ sudo ldconfig

Testing.
With the three packages installed, you can test GStreamer 1.18.4 on your Raspberry Pi 4. The screen dump below shows the installed version and a test pipeline $ gst-launch-1.0 videotestsrc ! videoconvert ! autovideosink to see if everything works.

Test_32

Unfortunately, the rpicamsrc module only works with a 32-bit operating system because the userland GPU interface does not fully support the new 64-bit operating system yet. Several modules have already been ported to 64 bits. However, the mmal core, which GStreamer needs, is still in the 32-bit version. We can still use GStreamer with all its features, but as source we need to use the old fashion /dev/video0 declaration.
32-bit OS testing.
The next test is to see the rpicamsrc working. If the $ gst-inspect-1.0 rpicamsrc command can be executed, it will certainly work correctly.

InspectRPiCam

Last test is the Raspicam projecting a preview on the screen by executing the following command $ gst-launch-1.0 -v rpicamsrc preview=true ! fakesink. Basically, the rpicamsrc has almost the same properties as the raspivid application. More information about the rpicamsrc can be found on the GStreamer site.
64-bit OS testing.
The 64-bit version uses the v4l2src module. First, see if the module is correctly installed with the command $ gst-inspect-1.0 v4l2src.

InspectV4L2Cam

If the Raspicam is connected and enabled in the Raspberry Pi Configurations, you will get the following screen when probing /dev/video0, indicating the camera is working.

InspectDevCam

TCP and UDP streaming.
The streaming commands are identical to those in version 1.14.4. If you have a 32 bit OS you can apply the rpicamsrc otherwise /dev/video0 will be used.
Cleaning.
Once you are certain GStreamer is working well, you can remove all the code and zip files.


# remove the core code and zip files
$ rm gstreamer-1.18.4.tar.xz
$ rm -rf gstreamer-1.18.4
# remove the plugins
$ rm -rf gst-plugins*
OpenCV.
For OpenCV to work with GStreamer 1.18.4, it must be recompiled, even if OpenCV was built with GStreamer 1.14. An OpenCV with version 1.14 will not work with the newly installed 1.18 version.
Please follow the steps given in our tutorials. The only difference is setting the -D WITH_GSTREAMER=ON argument in the build.
$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
        -D CMAKE_INSTALL_PREFIX=/usr/local \
        -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
   ..................        

        -D WITH_GSTREAMER=ON \

        ..................        

        -D OPENCV_GENERATE_PKGCONFIG=ON \
        -D BUILD_EXAMPLES=OFF ..


OpenCV_GStreamer

If the generated build information shows the detected GStreamer version, you can be confident OpenCV will support the new version once build and install. On our GitHub page you can find a simple GStreamer example with the Raspicam for a Raspberry Pi 4 32 or 64-bit OS.
WebCam.
GStreamer can also work with a USB webcam. There are two points to keep in mind.

First, there is the address of the camera. The Raspicam will always occupy /dev/video0 once enabled. All other USB cameras use the following numbers. So the first USB camera has /dev/video1, the next one /dev/video2 and so on. If there is no Raspicam enabled, then, of course, the numbering starts at /dev/video0.

The second point is the available output formats the USB webcam supports. Below an example of a popular Logitech c720 webcam is shown.

Logitech_G1

Logitech_G2

Logitech_G3

The source format is declared in the pipeline. It is the first part of the command, starting with video/x-raw, width=1280, height=720, framerate=30/1. Choose only a format supported by your webcam. In the list of the Logitech c720, you will find formats such as 640x480@30 FPS or 1280x960@5 FPS. Other formats give an error. The same goes for the MJPG formats. We specified that the source is video/x-raw, not video/x-mpeg. Using the MPEG stream is more for advanced users. Especially on the limited resources of the Raspberry Pi 4.

Streaming examples.

All samples are suitable for the 32- and 64-bit Raspberry operating system. We don't use the rpicamsrc as found on the 32-bit version to be as generic as possible. You can always replace v4l2src device=/dev/video0 with rpicamsrc in your pipeline to use the rpicamsrc if you want.
We have used the Raspicam V2 (IMX219) camera. Of course, you need to enable the camera first in the settings. As explained above, the pipelines can also be applied to other cameras, such as webcams, if you take care of the resolution and-or other properties.
Streaming to screen.
The first example is a simple streaming to the screen. You have seen it already working in the above sections. Note the scaling at the end of the pipeline.

for Bullseye replace v4l2src device=/dev/video0 with libcamerasrc
$ gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw, width=1280, height=720, framerate=30/1 ! videoconvert ! videoscale ! clockoverlay time-format="%D %H:%M:%S" ! video/x-raw, width=640, height=360 ! autovideosink

Streaming to file.
Another commonly used option of gstreamer is streaming to a file. Here are some examples.
The omxh264 plugin has been depreciated lately, not only because it will only work on a 32-bit operating system.
It is better to use encoding plugins that are more generic to the v4l2 Linux video engine, such as the v4l2h264enc plugin.
As of GStreamer version 1.18.4, some changes have been made to the initialization negotiation. You must explicitly specify the h264 encoding level now ('video/x-h264, level=(string)4'). In previous versions, the level was negotiated behind the scenes.
The video_bitrate parameter allows you to compromise between file size and amount of pixelation when scenes change quickly. Don't stress the pipeline too much with high resolutions, framerates or bitrates. Please note also the -e. This option lets the file close gracefully when the pipeline stops. In other words, it prevents a corrupted file end when the pipe suddenly closes.
Final remark, the SD card on your Raspberry Pi will wear out quickly if you start writing video files continuously. It is better to write them on a USB stick or other media.

for Bullseye replace v4l2src device=/dev/video0 with libcamerasrc
# mkv (only version 1.14.4 on a 32 bit OS)
$ gst-launch-1.0 -e v4l2src device=/dev/video0 ! image/jpeg, width=1280, height=720, framerate=30/1 ! jpegdec ! omxh264enc ! video/x-h264, profile=high ! h264parse ! matroskamux ! filesink location=output.mkv

# mkv
$ gst-launch-1.0 -e v4l2src device=/dev/video0 ! video/x-raw,width=1280, height=720, framerate=15/1 ! v4l2h264enc extra-controls="controls, h264_profile=4, video_bitrate=620000" ! 'video/x-h264, profile=high, level=(string)4' ! h264parse ! matroskamux ! filesink location=output.mkv

# mp4
$ gst-launch-1.0 -e v4l2src device=/dev/video0 ! video/x-raw,width=1280, height=720, framerate=30/1 ! v4l2h264enc extra-controls="controls, h264_profile=4, video_bitrate=620000" ! 'video/x-h264, profile=high, level=(string)4' ! h264parse ! mp4mux ! filesink location=video.mp4

# flv
$ gst-launch-1.0 -e v4l2src device=/dev/video0 ! video/x-raw,width=1280, height=720, framerate=15/1 ! v4l2h264enc extra-controls="controls, h264_profile=4, video_bitrate=620000" ! 'video/x-h264, profile=high, level=(string)4' ! h264parse ! queue ! flvmux name=mux ! filesink location=video.flv

# flv (only on a 32 bit OS)
$ gst-launch-1.0 -e v4l2src device=/dev/video0 ! video/x-raw, width=1280, height=720, framerate=30/1 ! videoconvert ! omxh264enc ! video/x-h264 ! h264parse ! video/x-h264 ! queue ! flvmux name=mux ! filesink location=video.flv

# mkv (only version 1.14.4 - missing 'video/x-h264, level=(string)4' used for 1.18.4)
$ gst-launch-1.0 -e v4l2src device=/dev/video0 ! video/x-raw, width=1920, height=1080, framerate=15/1 ! v4l2h264enc extra-controls="controls, h264_profile=4, video_bitrate=620000" ! h264parse ! matroskamux ! filesink location=output.mkv

# the last example is how to stream to screen with v4l2h264enc
$ gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,width=1280, height=720, framerate=15/1 ! v4l2h264enc ! 'video/x-h264, level=(string)4' ! decodebin ! videoconvert ! autovideosink


Streaming to OpenCV.
The following example is streaming to your OpenCV application. The best practice is by using raw images instead of motion compressed streams like mp4. The sink is now called appsink. The pipeline is encapsulated in a single routine. We show you a snippet of code, more on our GitHub page.


#include <opencv2/opencv.hpp>

std::string gstreamer_pipeline(int device, int capture_width, int capture_height, int framerate, int display_width, int display_height) {
    return
            " v4l2src device=/dev/video"+ std::to_string(device) + " !"
            " video/x-raw,"
            " width=(int)" + std::to_string(capture_width) + ","
            " height=(int)" + std::to_string(capture_height) + ","
            " framerate=(fraction)" + std::to_string(framerate) +"/1 !"
            " videoconvert ! videoscale !"
            " video/x-raw,"
            " width=(int)" + std::to_string(display_width) + ","
            " height=(int)" + std::to_string(display_height) + " ! appsink";
}
////////////
int main()
{
    //pipeline parameters
    int capture_width = 1280 ;
    int capture_height = 720 ;
    int display_width = 640 ;
    int display_height = 360 ;
    int framerate = 30 ;

    std::string pipeline = gstreamer_pipeline(0,capture_width, capture_height,
                                              display_width, display_height, framerate);

    cv::VideoCapture cap(pipeline, cv::CAP_GSTREAMER);
    if(!cap.isOpened()) {
        std::cout << "Failed to open camera." << std::endl;
        return (-1);
    }
UDP streaming.
For completeness, the UDP streamer again. The port number (5200) is arbitrary. You can choose any number you want, preferably over 1000, as long as the sender and receiver use the same number.

for Bullseye replace v4l2src device=/dev/video0 with libcamerasrc
# get the IP address of the recieving RPi first
$ hostname -I
# start the sender, the one with the Raspicam
$ gst-launch-1.0 -v v4l2src device=/dev/video0 num-buffers=-1 ! video/x-raw, width=640, height=480, framerate=30/1 ! videoconvert ! jpegenc ! rtpjpegpay ! udpsink host=192.168.178.84 port=5200
# start the reciever, the one with IP 192.168.178.84
$ gst-launch-1.0 -v udpsrc port=5200 ! application/x-rtp, media=video, clock-rate=90000, payload=96 ! rtpjpegdepay ! jpegdec ! videoconvert ! autovideosink

TCP streaming.
And the TCP streamer once more. Same as with UDP, you can choose any port you like.

for Bullseye replace v4l2src device=/dev/video0 with libcamerasrc
# get the IP address of the sending RPi first
$ hostname -I
# start the sender, the one with the Raspicam and IP 192.168.178.32
$ gst-launch-1.0 -v v4l2src device=/dev/video0 num-buffers=-1 ! video/x-raw,width=640,height=480, framerate=30/1 ! videoconvert ! jpegenc ! tcpserversink  host=192.168.178.32 port=5000
# start the reciever and connect to the server with IP 192.168.178.32
$ gst-launch-1.0 tcpclientsrc host=192.168.178.32 port=5000 ! jpegdec ! videoconvert ! autovideosink

RTSP streaming.
RTSP streaming is widespread. It is designed to control media streaming sessions between endpoints. In contrast to the single client connection of TCP and UDP, RTSP can connect a single server to multiple clients. In practice, the number of clients will be limited by the bandwidth capacity of a Raspberry Pi.
Before you can start streaming RTSP, you need the gst-rtsp-server and its examples. See the installation instructions in the section of your GStreamer version 1.14.4, or 1.18.4. As you can see, the pipeline assumes that your source is capable of the x-h264 format, like the raspicam. If not, you need to convert the format.


Buster 32 or 64-bits OS
Bullseye 32 or 64-bits OS
# select the proper folder
$ cd ~/gst-rtsp-server-1.18.4/build/examples
# run the pipeline
$ ./test-launch "libcamerasrc ! video/x-h264, width=640, height=480, framerate=30/1 ! h264parse config-interval=1 ! rtph264pay name=pay0 pt=96"
rpicamsrc
You can receive the stream with the VLC player for example. In the Media menu, select the option Open Network Steam... and enter the address of your Raspberry Pi (in our case 192.168.178.32) followed by the postfix :8554/test as you can see in the screen dump below.

 VLC menu

VLC media player

If you like to use GStreamer to receive the stream, use the following command.


# use the IP address of the sending RPi (192.168.178.32) first
$ gst-launch-1.0 rtspsrc location=rtsp://192.168.178.32:8554/test/ latency=10 ! decodebin ! autovideosink
Raspberry Pi to YouTube live streaming.
RTMP (Real-Time Messaging Protocol) streaming is a TCP-based protocol used by YouTube, among others. It is possible to stream RTMP from one Raspberry Pi to another. In that case, you need a server like Nginx to facilitate all network protocols. It is beyond the scope of this article.
We'll start with some steps in YouTube Studio to get your Raspberry Pi stream to a YouTube live channel.
YouTube6.png
❮❯
       
The very first step is to create an account on YouTube, if you don't have one yet.
In YouTube studio, go to your dashboard and select create (1), go live (2). The very first time you want to stream, it will take 24 hours for YouTube to activate your account for live streaming.
You want to start right now, no need for scheduling your stream.
Select the streaming software option. We are using the Raspberry Pi or Jetson Nano instead of the webcam.
You probably don't make the stream for children, so 'no' is your answer here.
Now you get a most important page. Here you will find the IP address and your private key, both generated by YouTube. Your key will remain active for as long as you wish. Even after a week of not streaming, it is still usable. You can think of it as your personal live streaming key. Both IP and Key must be specified in a GStreamer command, as shown in the slide.
Once you give the correct GStreamer command to your Raspberry Pi, you will see the stream in the preview after a few seconds.
At the same time, YouTube has published the live stream on your YouTube channel.
Once your stream stops for a few minutes, you will have to re-activate the live stream again at your dashboard. YouTube is not listening forever if your stream is active or not. You have to notify YouTube.

Not so obvious, but YouTube expects to find sound in the live stream in addition to images. Without sound, the stream will not be accepted. Because most Raspberry Pi cameras work without a microphone, we use a 'dummy' audio stream, the green part of the command line.
We need to send an h264 formatted video. This format is already available on the Raspberry PI camera, so you don't have to compose it, saving some CPU/GPU performance. On the other hand, you cannot manipulate the stream. For example, it is not possible to provide a timestamp. It requires individual images, not a motion compressed stream. The second command below gives you an impression of how you can accomplish this. Also, note the difference in resolution. You can choose most resolutions available on your Raspberry Pi camera or webcam.
Start live streaming from your Raspberry Pi camera with the following command.
If you have a 32-bit Raspberry OS and rpicamsrc is installed, you can replace v4l2src device=/dev/video0 with rpicamsrc preview=false.


for Bullseye replace v4l2src device=/dev/video0 with libcamerasrc
# get your IP address and key from YouTube first
$ gst-launch-1.0 v4l2src device=/dev/video0 ! \
  video/x-h264, width=1640, height=922, framerate=30/1 ! h264parse ! \
  mux. \
  audiotestsrc wave=silence ! voaacenc bitrate=128000 ! aacparse ! \
  mux. \
  flvmux streamable=true name=mux ! queue ! \
  rtmpsink location="rtmp://<youtube IP address>/<your key>"

# an example of a timestamp. note the h264 compression, done by CPU/GPU.
$ gst-launch-1.0 v4l2src device=/dev/video0 ! \
  video/x-raw,width=1280,height=720,framerate=15/1 ! \
  videoconvert ! videoscale ! clockoverlay time-format="%D %H:%M:%S" ! \
  v4l2h264enc extra-controls="controls, h264_profile=4, video_bitrate=620000" ! \
  h264parse ! \
  mux. \
  audiotestsrc wave=silence ! voaacenc bitrate=128000 ! aacparse ! \
  mux. \
  flvmux streamable=true name=mux ! queue ! \
  rtmpsink location="rtmp://<youtube IP address>/<your key>"





source : https://qengineering.eu/install-gstreamer-1.18-on-raspberry-pi-4.html
