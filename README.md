# How to run ROS Melodic on Ubuntu 20/21/22 with Singularity
- `$` before a command means to run in it in a normal terminal
- `Singularity>` before a command means to run it inside the singularity container


## Install Singularity
We will download the source code, compile it and then install Singularity.

Download the necessary libs to compile code:
```
$ sudo apt-get update

$ sudo apt-get install -y build-essential libssl-dev uuid-dev libgpgme11-dev \
    squashfs-tools libseccomp-dev wget pkg-config git cryptsetup debootstrap
```


Since Singuliarty is written in Go we need to install Go to compile it.
```
$ cd ~/Downloads

$ wget https://dl.google.com/go/go1.13.linux-amd64.tar.gz

$ sudo tar --directory=/usr/local -xzvf go1.13.linux-amd64.tar.gz

$ export PATH=/usr/local/go/bin:$PATH
```


Now we can download the Singularity source code.
```
$ wget https://github.com/singularityware/singularity/releases/download/v3.5.3/singularity-3.5.3.tar.gz

$ tar -xzvf singularity-3.5.3.tar.gz
```

And compile the code.
```
$ cd singularity

$ ./mconfig

$ cd builddir

$ make
```

And to install Singularity from the compiled code
```
$ sudo make install
```

To verify the installation of Singularity:
```
$ singularity run library://godlovedc/funny/lolcow
```

If the installation was succesful you should see something like this:
```
 ______________________________________
/ Q: What do they call the alphabet in \
\ Arkansas? A: The impossible dream.   /
 --------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

We don't need the Go and Singularity source doe anymore. If you want you can delete it from the `~/Downloads` folder.


## Create the Singularity image with ROS
We don't need to create the Singularity container from scratch. Instead we can use an existing Docker image with Ubuntu 18 and ROS Melodic as input.
```
$ sudo singularity build --sandbox melodic/ docker://osrf/ros:melodic-desktop-full
```

To run the newly created container:
```
$ sudo singularity shell --writable melodic/
```

Let's update the container packages first. (Run inside the Singularity).
```
Singularity> apt update && apt upgrade

```

If you now try to run Gazebo or any other GUI program inside the singuliarty you will probably get the following error:
```
No protocol specified
```

We first need to allow those programs access to render on the display. You can do this by executing the following command outside the Singularity, so inside a normal command prompt:
```
$ xhost +local:root
```

Okay now try to run Gazebo inside the Singularity:
```
Singularity> gazebo
```

If Gazebo works for you, you can go to the next session :-)

If you have the following error we need to update the graphics drivers inside the Singuliarty container.
```
X Error of failed request:  BadAlloc (insufficient resources for operation)
  Major opcode of failed request:  149 ()
  Minor opcode of failed request:  2
  Serial number of failed request:  35
  Current serial number in output stream:  36 
```

To update the drivers:
```
Singularity> apt install software-properties-common
Singularity> add-apt-repository ppa:kisak/kisak-mesa -y
Singularity> apt update
Singularity> apt upgrade -y
Singularity> apt --fix-broken install
```


## Setup the Tiago environment
To access the source code stored on GitLab we need to store our SSH key on the GitLab server.

To access your SSH key:
```
$ gedit ~/.ssh/id_rsa.pub
```

If this file does not exist, generate an SSH key with `ssh-keygen`, and just press `enter` a few times. Then add it with `ssh-add`.

Copy the whole key:
```
ssh-rsa 
---
---
---
---
pepijn@pepijn-thinkpad
```

and store it on the GitLab server under Preferences -> SSH Keys -> Add an SSH key

Now we can download the code. (Dont forget to change your team number inside the URL)

```
$ mkdir ~/mdp/src && cd ~/mdp/src
$ git clone git@gitlab.tudelft.nl:cor/ro47007/2022/team-9/cor_mdp_tiago.git
```

To clone the required dependencies `vcstool` is used. But first we need to install `vcstool`.
```
$ sudp apt install python3-pip
$ sudo pip install -U vcstool
```

Now import the dependencies using the `vcstool`
```
$ vcs import --input cor_mdp_tiago/cor_mdp_tiago.rosinstall .
```

Next we use `rosdep` to install the other dependencies from inside the singularity.

However the current Singularity container does not have access to our workspace. Exit the running Singuliarty container with `Ctrl + D`.

Now restart it with this command:  (don't forget to replace your username)
```
$ sudo singularity shell -B /home/pepijn/:/home/ -w melodic/
```

And activate the ROS environment
```
Singularity> source /opt/ros/melodic/setup.bash
```

Now inside the Singularity we go to the mdp folder:
```
Singularity> cd /home/mdp
```

And use `rosdep` to install the other dependencies:
```
Singularity> rosdep update
Singularity> rosdep install --from-paths src --ignore-src --rosdistro melodic -y --skip-keys="opencv2 opencv2-nonfree pal_laser_filters speed_limit_node sensor_to_cloud hokuyo_node libdw-dev python-graphitesend-pip python-statsd pal_filters pal_vo_server pal_usb_utils pal_pcl pal_pcl_points_throttle_and_filter pal_karto pal_local_joint_control camera_calibration_files pal_startup_msgs pal-orbbec-openni2 dummy_actuators_manager pal_local_planner gravity_compensation_controller current_limit_controller dynamic_footprint dynamixel_cpp tf_lookup opencv3 joint_impedance_trajectory_controller"
```

Before we can use the `catkin` command we need to install it:
```
Singularity> apt install python3-catkin-tools
```

And let's also install Numpy:
```
Singularity> apt install python3-numpy
```

Finally we can build the Tiago workspace:
```
Singularity> catkin build
```

And activate it
```
Singularity> source devel/setup.bash
```

To your project
```
Singularity> roslaunch cor_mdp_tiago_gazebo tiago_ahold.launch # Ahold
Singularity> roslaunch cor_mdp_tiago_gazebo tiago_festo.launch # Festo
```

You are done!

## References
- https://singularity-tutorial.github.io/01-installation/
- https://blog.sravjti.in/2020/12/11/singularity-ros-melodic.html








