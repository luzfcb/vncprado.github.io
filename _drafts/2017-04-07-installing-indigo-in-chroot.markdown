---
layout: post
title: "installing indigo in chroot"
date: "2017-04-07 15:09:47 -0400"
---


This is Google's cache of http://wiki.ros.org/ROS/Tutorials/InstallingIndigoInChroot. It is a snapshot of the page as it appeared on Apr 5, 2017 20:04:05 GMT.
The current page could have changed in the meantime. Learn more
Full versionText-only versionView sourceTip: To quickly find your search term on this page, press Ctrl+F or ⌘-F (Mac) and use the find bar.
ros.org	About | Support | Status | answers.ros.org
Search:
DocumentationBrowse SoftwareNewsDownload
ROS
Tutorials
InstallingIndigoInChroot
Wiki

Distributions
ROS/Installation
ROS/Tutorials
RecentChanges
InstallingIndigoInChroot
Page

Immutable Page
Info
Attachments
More Actions: Raw Text Print View Render as Docbook Delete Cache ------------------------ Check Spelling Like Pages Local Site Map ------------------------ Rename Page Copy Page Delete Page ------------------------ My Pages Subscribe User ------------------------ Remove Spam Revert to this revision Package Pages Sync Pages ------------------------ CreatePdfDocument Load RawFile Save SlideShow
User

Login
(!) Please ask about problems and questions regarding this tutorial on answers.ros.org. Don't forget to include in your question the link to this page, the versions of your OS & ROS, and also add appropriate tags.
Installing ROS Indigo in a chroot

Description: This tutorial walks you through installing ROS Indigo (and Ubuntu 14.04) in a chroot. This allows you to have a ROS Indigo installation even on Ubuntu versions that Indigo doesn't build on.

Tutorial Level: INTERMEDIATE

Contents

Introduction
Creating the Ubuntu 14.04 Chroot
Setup ROS Indigo in the Chroot
Compiling Source Packages
Running Graphical Programs
Accelerated OpenGL
NVIDIA
Testing
Introduction

Because ROS Indigo is being released at the same time as a new Ubuntu long term support version (Trusty Tahr), Indigo and Hydro have no overlap in terms of Ubuntu distributions. For development, it is useful to be able to install both Indigo and Hydro on the same machine, and incrementally port your code over.

Chroot (change root) makes this possible. A chroot is an directory on your system which has an entire Ubuntu in it. When you are in the chroot, your root directory / actually points to some subdirectory on the overall system. In this tutorial, that subdirectory will be /srv/chroot/indigo_trusty. Since that will look like the root directory to programs inside the chroot, when the look for /etc for example, they'll end up in /srv/chroot/indigo_trusty/etc.

The debootstrap tool is the second piece of this. It will create a whole new apt (the Debian package manager) configuration inside of the chroot. We'll set this up to use the 14.04 Ubuntu repositories, so that when we install ROS it will get all the dependencies from 14.04, and install everything to /srv/chroot/indigo_trusty.

This tutorial has been tested on the following systems:

Ubuntu Xenial (16.04): 64 bit
Ubuntu Precise (12.04): 32 and 64 bit
Ubuntu Raring (13.04): 64 bit
Creating the Ubuntu 14.04 Chroot

First install schroot and debootstrap:

sudo apt-get install debootstrap schroot
Next create a config file for the chroot we're making:

sudo gedit /etc/schroot/chroot.d/indigo_trusty.conf
Copy the following text into the config file:

[indigo_trusty]
description=Ubuntu 14.04 for installing ROS Indigo
directory=/srv/chroot/indigo_trusty
root-groups=root
type=directory
users=USERNAME
personality=linux
preserve-environment=false
Change USERNAME to your own user name.


Create the directory where we'll put the chroot:

sudo mkdir -p /srv/chroot/indigo_trusty
Run debootstrap to setup the new chroot (this assumes you are on a 64 bit system. If you are on a 32 bit system, change amd64 to i386):

sudo debootstrap --variant=buildd --arch=amd64 trusty /srv/chroot/indigo_trusty http://archive.ubuntu.com/ubuntu/
List the chroots:

schroot -l
If everything has worked so far, you should see indigo_trusty listed.

Switch to the new chroot. You may get some errors here if you have stuff in your .bashrc that uses programs not yet installed in the chroot, but you can (usually) ignore them. We use sudo here so that we'll end up with a root shell inside of the chroot, and we'll be able to install packages:

sudo schroot -c indigo_trusty
You're now inside of a chroot environment. If you run ls /, you'll actually see the root directory of the chroot environment, which is /srv/chroot/indigo_trusty on the host machine.

Install some useful packages:

apt-get install sudo vim git
Now that we've installed sudo in the chroot, we don't need to run schroot -c with sudo anymore. Exit the chroot and re-enter it without using sudo:

exit
schroot -c indigo_trusty
Some programs need to know the locale:

sudo locale-gen en_US en_US.UTF-8
sudo dpkg-reconfigure locales
Setup ROS Indigo in the Chroot

ROS needs some dependencies from the Ubuntu universe repo:

sudo sh -c 'echo "deb http://archive.ubuntu.com/ubuntu trusty universe" >> /etc/apt/sources.list'
Add the ROS repo:

This command will add the "shadow fixed" ROS repository, which contains brand new builds and is not as stable as the standard ROS repository. While Indigo is still brand new, we need ros-shadow-fixed in order to get packages that were just released. If you want a more stable system, however, you should replace "ros-shadow-fixed" with "ros".


sudo sh -c 'echo "deb http://packages.ros.org/ros-shadow-fixed/ubuntu trusty main" > /etc/apt/sources.list.d/ros-latest.list'
Import the ROS package repository key:

sudo apt-get install wget
wget http://packages.ros.org/ros.key -O - | sudo apt-key add -
Update the list of packages:

sudo apt-get update
Now if you run apt-cache search ros-indigo, you should see the packages that have already been released into Indigo.

Install rosdep; we'll use this to install dependencies of source packages that we are working on:

sudo apt-get install python-rosdep
sudo rosdep init
rosdep update
Compiling Source Packages

We'll compile the roscpp tutorials as an example.

Your home directory is shared between the main system and the chroot, so you can use editors outside the chroot to edit code if you want. For this tutorial, however we'll assume that you're doing all your editing inside the chroot.


Create a catkin workspace:

mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
git clone https://github.com/ros/ros_tutorials.git
cd ~/catkin_ws
Use rosdep to install the dependencies:

rosdep install --from-path src --rosdistro indigo -i -y
Build the catkin workspace:

source /opt/ros/indigo/setup.bash
catkin_make
Install rosbash (for things like rosrun):

sudo apt-get install ros-indigo-rosbash
Start a roscore:

schroot -c indigo_trusty
source /opt/ros/indigo/setup.bash
roscore
If the roscore does not start up cleanly, make sure there is not one already running on your host computer. Then, verify that the ROS_MASTER_URI environment variable is set correctly. The usual setting should work:

export ROS_MASTER_URI=http://$HOSTNAME:11311/


In a separate terminal, enter the chroot, and run the talker node:

source devel/setup.bash
rosrun roscpp_tutorials talker
In a third terminal, enter the chroot, and run the listener node:

source devel/setup.bash
rosrun roscpp_tutorials listener
You should see output similar to this one:

[ INFO] [1396326497.204811314]: I heard: [hello world 135]
[ INFO] [1396326497.303949602]: I heard: [hello world 136]
Running Graphical Programs

In order to use graphical programs from the Indigo chroot you will need to set the DISPLAY environment variable and allow access to the host X server.

In the host environment:

xhost +local:
This command allows non-network local connections access to your host environment's X server, which should be ok. For more information, see the xhost man page.


Export the DISPLAY variable in the chroot:

schroot -c indigo_trusty
export DISPLAY=:0
In the same terminal, try opening turtlesim to test it:

source ~/catkin_ws/devel/setup.bash
rosrun turtlesim turtlesim_node
The turtlesim window should appear. You may run (in a separate terminal) the turtlesim turtle_teleop_key to move the turtle.

Accelerated OpenGL

In order to be able to run RViz (or any other program that needs accelerated OpenGL), the OpenGL client library files will need to be installed inside the chroot.

NVIDIA

It is important for the versions of the libraries installed in the chroot to be as close as possible (ideally: identical) to those installed in the host environment. To find the version of the NVIDIA driver installed:

dpkg -l | grep -i nvidia
Now go to the NVIDIA site and download the binary drivers closest to the one installed on the host (be sure to download the appropriate drivers: x86_64 for 64 bit systems, x86 for 32 bit).

Copy them to the chroot (place them in /path/to/my/chroot/tmp or something similar).

As we only require the OpenGL libraries to be installed, the following command should suffice:

schroot -c indigo_trusty
sudo sh /tmp/NVIDIA-Linux-x86_64-331.20.run --no-precompiled-interface --no-runlevel-check --no-kernel-module --no-x-check --no-kernel-module-source --no-distro-scripts --no-nvidia-modprobe --accept-license
This example installs the 64 bit version of the 331.20 drivers. Change this to match the actual version you are installing.


The installer will probably display some errors (ERROR: Unable to create ‘(null)’ for copying (Bad address)): these can be ignored (press enter).

Testing

Use glx-gears to test the OpenGL setup:

schroot -c indigo_trusty
sudo apt-get install mesa-utils
glxgears -info
You should see the spinning gears, and the console should show output similar to:

Running synchronized to the vertical refresh.  The framerate should be
approximately the same as the monitor refresh rate.
GL_RENDERER   = GeForce [..]
GL_VERSION    = X.Y.Z NVIDIA 331.20
GL_VENDOR     = NVIDIA Corporation
GL_EXTENSIONS = GL_ARB_arrays_of_arrays [..]
This output is obviously different for AMD/Intel/etc graphic cards, but it should still display the proper values for the reported variables.


Wiki: ROS/Tutorials/InstallingIndigoInChroot (last edited 2016-07-09 16:09:30 by JackOQuin)

Except where otherwise noted, the ROS wiki is licensed under the
Creative Commons Attribution 3.0 | Find us on Google+
