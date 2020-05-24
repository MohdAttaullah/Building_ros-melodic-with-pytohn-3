# Building_ros-melodic-with-pytohn-3

These Steps Assumes few things:

You are familiar with ROS
You would like to run ROS Melodic in Ubuntu Linux 18.04 with Python3 support
You are familiar with the ROS build system
Familiar with the Linux command line, a shell like bash, and an editor like vim

Remove all things python2 (Optional)
This step is optional, but I recommend it to have a clean build. Any other system packages that are removed can be reinstalled later by following the instructions here. So, to remove all python2 packages, execute the following on the command line:

```sudo apt-get remove python-*```

Remove any previous installations of ROS
It’s probably a good idea to remove any previous versions of ROS. So, for example, to remove a default melodic install, execute the following in a shell:

```
sudo apt-get remove ros-*
sudo apt-get remove ros-melodic-*
sudo apt-get autoremove
```

Setup Python3 dependencies
Since we will be building ROS Melodic from source to support python3, we need to install several dependencies.

sudo apt update
sudo apt install -y python3 python3-dev python3-pip build-essential

We’ll also need to install ROS specific packages using pip3:

sudo -H pip3 install rosdep rospkg rosinstall_generator rosinstall wstool vcstools catkin_tools catkin_pkg

Initialize catkin build environment
As with any standard ROS installation, we need to initialize rosdep:

sudo rosdep init
rosdep update

and create the catkin workspace that we will use to build ROS. Please be sure of the location of the workspace, since the entire build will be tied to that location. If the workspace folder is deleted or moves, it will corrupt the installation.

cd ~
mkdir ros_catkin_ws
cd ros_catkin_ws

Next, we have to initialize the catkin workspace. If you’re unfamiliar with the catkin build tools, you can check out the docs here. Note: This initialization will build the release version of ROS Melodic and install it to a directory called “install” that is located within the ros_catkin_ws directory.

catkin config --init -DCMAKE_BUILD_TYPE=Release --blacklist rqt_rviz rviz_plugin_tutorials librviz_tutorial --install

Setup ROS install
ROS has many different flavors of installations: desktop, desktop-full, ros_core, robot, etc. For development purposes, I would recommend installing desktop-full, but feel free to install the flavor that meets your needs. Simply replace desktop-full with the installation flavor of your choice, e.g. ros_base. You can read more about the various options here.

rosinstall_generator desktop_full --rosdistro melodic --deps --tar > melodic-desktop-full.rosinstall
wstool init -j8 src melodic-desktop-full.rosinstall

If a download fails, just re-run the following command until all packages download properly:

wstool update -j4 -t src

Setup environment and install dependencies
Here is where things start to diverge a bit from the default build from source procedure. First, we need to have an environment variable called ROS_PYTHON_VERSION set to 3:

export ROS_PYTHON_VERSION=3

We need to install wxPython for the build to complete.

pip3 install -U -f https://extras.wxpython.org/wxPython4/extras/linux/gtk3/ubuntu-18.04 wxPython

We will also need to create a shell script called install_skip with the following:

#!/bin/bash
#Check whether root
if [ $(whoami) != root ]; then
    echo You must be root or use sudo to install packages.
    return
fi

#Call apt-get for each package
for pkg in "$@"
do
    echo "Installing $pkg"
    sudo apt-get -my install $pkg >> install.log
done

and make the script executable:

chmod +x install_skip

And now for a little SED magic!
We need to install all of the dependencies for the ROS source, but for python3 instead of python3. That means that we need to see what the ROS packages need and install the python3 version instead of the python2 version. We can do that with the following:

sudo ./install_skip `rosdep check --from-paths src --ignore-src | grep python | sed -e "s/^apt\t//g" | sed -z "s/\n/ /g" | sed -e "s/python/python3/g"`

We’ll now install all of the remaining dependencies using rosdep and skip the python2 based dependencies:

rosdep install --from-paths src --ignore-src -y --skip-keys="`rosdep check --from-paths src --ignore-src | grep python | sed -e "s/^apt\t//g" | sed -z "s/\n/ /g"`"

The final step before building is replacing all references to python2 in the shebangs to python3:

find . -type f -exec sed -i 's/\/usr\/bin\/env[ ]*python/\/usr\/bin\/env python3/g' {} +

Be sure to only run this once. If you run this twice by mistake, you might end up with shebangs that have python33 instead of python3.

Finally, build ROS!
With all the setup out of the way, we’re ready to build ROS Melodic with python3 support:

catkin build

And that’s it! Once the build finishes, you will have a ROS Melodic build that supports python3. The last step is to export the PYTHON_PATH environment variable to let ROS know where all of the system python3 packages are installed.

export PYTHONPATH=/usr/lib/python3/dist-packages  

You should do this prior to sourcing the setup in the devel directory:

source ~/ros_catkin_ws/install/setup.bash

to be able to access ROS via the command line (assuming that you are running bash).

In summary, assuming that you are familiar with Ubuntu, the command line, and the ROS build system, I showed you how to build ROS Melodic with python3 support.
