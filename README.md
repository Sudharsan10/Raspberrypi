# Raspberrypi for Robotics
Setting up Raspberry Pi 4 model B for typical robotics application
## Instructions to set up Raspberry Pi for sensor Integration

## Install Ubuntu Server
- Install [Ubuntu 20.04.2 Server](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#1-overview)
using this official guide.
### 1.1 Before first boot
After successfully itching the ubuntu image to the microSD card, lets set up the network credentials for the pi
- Edit `network-config` file in the root directory of the SD card.
- You will find section commented like below
    ```
    #wifis:
    #  wlan0:
    #    dhcp4: true
    #    optional: true
    #    access-points:
    #      myhomewifi:
    #        password: "S3kr1t"
    #      myworkwifi:
    #        password: "correct battery horse staple"
    #      workssid:
    #        auth:
    #          key-management: eap
    #          method: peap
    #          identity: "me@example.com"
    #          password: "passw0rd"
    #          ca-certificate: /etc/my_ca.pem
    ```
- Replace the above with your network details and uncomment the necessary lines, for example:
    ```      
    wifis:
      wlan0:
        dhcp4: true
        optional: true
        access-points:
          myhomewifi:
            password: "S3kr1t"
          myworkwifi:
            password: "correct battery horse staple"
    #      workssid:
    #        auth:
    #          key-management: eap
    #          method: peap
    #          identity: "me@example.com"
    #          password: "passw0rd"
    #          ca-certificate: /etc/my_ca.pem
    ```
    Note: make sure to use proper indentation.

### 1.2 First boot
- Plug in a monitor, keyboard, mouse and plug the USB-C cable to the RPi4
- After successful boot up, enter the following default credentials to login into the RPi4
    - Username: `ubuntu`
    - Password: `ubuntu`
- Change the default password
- run the command 
  > ```
  > sudo reboot
  > ```
 
### 1.3 After First boot
- Once you log in with you new credentials you will find the IP of the Pi being printed in the console.
- Note down the IP for ssh into RPi4 in the future
- Check for internet connectivity by running the command `ping google.com`
    > ```
    > ubuntu@ubuntu $: ping google.com
    > PING google.com (172.217.15.78) 56(84) bytes of data.
    > 64 bytes from iad23s63-in-f14.1e100.net (172.217.15.78): icmp_seq=1 ttl=113 time=4.56 ms
    > 64 bytes from iad23s63-in-f14.1e100.net (172.217.15.78): icmp_seq=2 ttl=113 time=11.8 ms
    > 64 bytes from iad23s63-in-f14.1e100.net (172.217.15.78): icmp_seq=3 ttl=113 time=4.56 ms
    > ```
- Now lets update the ubuntu by running the following commands
    > ```
    > sudo apt-get update
    > sudo apt-get upgrade -y
    > ```
## Install Ubuntu Desktop
 
- For VNC server this section is a requirement.
- Run the following command to install the Ubuntu Desktop.
```
sudo reboot #optional
sudo apt-get install ubuntu-desktop
```

## Change local host name & Add new users with sudo permission  
### 2.1 Change hostname
- To change the static host name run the following command
    > `sudo hostnamectl set-hostname mr.robot`
- Optionally you can also set the pretty hostname:
    > `sudo hostnamectl set-hostname "mr.robot's robot" --pretty`
    >
    Note: You can also change hostname by modifying the files eg.,  modify the file `/etc/hostname`
by replacing the content with the hostname of your choice
and modify the file `/etc/machine-info` to set pretty hostname.
- To verify the hostname has been fully changed, enter the hostnamectl command:
    > `hostnamectl`
### 2.2 Add new user and change the default user 'Ubuntu'
- To create a new user account named username you would run:
    > `sudo adduser username`
- Remove user by running following command
    > `sudo deluser username` 
    
    or to remove user along with its home directory 
    
    > `sudo deluser --remove-home username`
  
    Note: Log in to root user or different user to delete a user.
- Add `sudo` privilege to new users created by running the following command,
    > `sudo usermod -aG sudo username`
  
    Note: To add `sudo` privileges to the currently logged-in user, firs tloggout and 
log into either root user or user with `sudo` privileges.  
- Verify create user has `sudo` privileges by running following command,
    > `su - username`

### 2.3 Add/remove user to user groups
- To add user to a particular user group,
    > `sudo adduser username usergroup`
- So to add 'newuser' to the group 'dialout' type:
    > `sudo adduser newuser dialout`  
- Also to remove user 'newuser' from the group 'adm' you would type:
    > `sudo deluser newuser adm`

Note: after adding user to a user group, please logout and log in back for the changes to 
be applied.

## Enable access to ports
### 3.1 Serial Ports
- Add user to dialout group for accessing Serial ports
    > `sudo usermod -a -G dialout username`
### 3.2 Disable UART during boot sequence.
When connecting sensor devices like GPS to the UART, it may hinder the pi during boot up triggering U-boot sequence since 
any data through UART is consider as a keyboard interrupt and leads to U-boot sequence, so we need to disable the UART during boot.
- you can either copy the precompiled and modified `uboot_rpi_4.bin` 

### 3.3 Fix for RPi won't bootup without HDMI connected.
Go to `/boot/firmware/usercfg.txt` and add the below line to the file 
```
hdmi_force_hotplug=1

# optional
[HDMI:0]
hdmi_group=2
hdmi_mode=82
hdmi_drive=2
```
Upon boot up you should have no issue even if HDMI is not plugged in.
### 3.4 Turn Wifi Powermanagement off (Optional)
```
sudo iwconfig wlan0 power off
```


## ROS Noetic Installation.
Go to this link [ROS Noetic installation](http://wiki.ros.org/noetic/Installation)
and follow the official guide to install ROS in ubuntu or follow the below steps.

### 4.1 Setup your source.list
```
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
```
### 4.2 Setup your keys
```
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
```

### 4.3 ROS Noetic Desktop Installation
- First, make sure your Debian package index is up-to-date,
  ```
  sudo apt update
  ```
  
- Install Ros Desktop full:
  ```
  sudo apt install ros-noetic-desktop-full
  ```

- To use ROS in your terminal source the ROS installation,
  ````
  source /opt/ros/noetic/setup.bash
  ````
### 4.4 ROS Dependencies for building packages
```
sudo apt install python3-rosdep python3-rosinstall python3-rosinstall-generator python3-wstool build-essential
```

### 4.5 Initialize rosdep
Before you can use many ROS tools, you will need to initialize rosdep. rosdep enables you to easily install system dependencies for source you want to compile and is required to run some core 
components in ROS. If you have not yet installed rosdep, do so as follows.
```
sudo apt install python3-rosdep
```
With the following, you can initialize rosdep.
```
sudo rosdep init
rosdep update
```

### 4.6 Pip3 install
```
sudo apt install python3-pip
```
## Useful tools
```
sudo apt-get install wireless-tools
sudo apt-get install net-tools
```
## References
1. Install [Ubuntu 20.04.2 Server](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#1-overview)
2. Change [hostname](https://linuxize.com/post/how-to-change-hostname-on-ubuntu-20-04/) in ubuntu 20.04.2
3. Add/ delete [new user](https://linuxize.com/post/how-to-add-and-delete-users-on-ubuntu-20-04/) in ubuntu 20.04.2
4. Add/remove [username to user group](https://thepihut.com/blogs/raspberry-pi-tutorials/how-to-add-new-users-and-add-users-to-groups) in ubuntu 20.04.2
5. [Disabling UART during boot sequence](https://askubuntu.com/questions/1215848/how-to-disable-ttyama0-console-on-boot-raspberry-pi)
6. Install [ROS Noetic](http://wiki.ros.org/noetic/Installation) using offical guide.
    