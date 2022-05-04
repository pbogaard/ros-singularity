# How to run ROS Melodic on Ubuntu 20/21/22 with Singularity

## Install Singularity
We will download the source code, compile it and then install Singularity.

Download the necessary libs to compile code:
```
sudo apt-get update

sudo apt-get install -y build-essential libssl-dev uuid-dev libgpgme11-dev \
    squashfs-tools libseccomp-dev wget pkg-config git cryptsetup debootstrap
```


Since Singuliarty is written in Go we need to install Go to compile it.
```
cd ~/Downloads

wget https://dl.google.com/go/go1.13.linux-amd64.tar.gz

sudo tar --directory=/usr/local -xzvf go1.13.linux-amd64.tar.gz

export PATH=/usr/local/go/bin:$PATH
```


Now we can download the Singularity source code.
```
wget https://github.com/singularityware/singularity/releases/download/v3.5.3/singularity-3.5.3.tar.gz

tar -xzvf singularity-3.5.3.tar.gz
```

And compile the code.
```
cd singularity

./mconfig

cd builddir

make
```

And to install Singularity from the compiled code
```
sudo make install
```

To verify the installation of Singularity:
```
singularity run library://godlovedc/funny/lolcow
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
sudo singularity build --sandbox melodic/ docker://osrf/ros:melodic-desktop-full
```

To run the newly created container:
```
sudo singularity shell --writable melodic/
```

Let's update the container packages first. (Run inside the Singularity).
```
apt update && apt upgrade

```

If you now try to run Gazebo or any other GUI program inside the singuliarty you will probably get the following error:
```
No protocol specified
```

We first need to allow those programs access to render on the display. You can do this by executing the following command outside the Singularity, so inside a normal command prompt:
```
xhost +local:root
```

Okay now try to run Gazebo inside the Singularity:
```
gazebo
```











