___
# toddcore

#### A complete, ready-to-load Raspberry Pi Zero W Raspbian image with ROS Melodic.  Set up as a ROS Appliance in support of Todd the Quad Type 2/3/S.  
Just [write the image to a MicroSD card](https://www.raspberrypi.org/documentation/installation/installing-images/README.md "write the image to a MicroSD card") and boot it up.  Roscore runs on the Raspberry Pi Zero W as master.  From there you can connect to it from any windows / linux / mac machine with as a [remote master](http://wiki.ros.org/ROS/Tutorials/MultipleMachines "emote master").

![toddcore htop](https://i.imgur.com/kwwD4tm.png "toddcore htop")

##### Everything takes about 3 minutes to boot, but once it does you have:
 - [Raspbian Buster Lite Image](https://www.raspberrypi.org/downloads/raspbian/ "Raspbian Buster Lite Image") from 2019-09-26
 - All raspian updates installed as of 2019-12-26
 - [ROS Melodic](http://wiki.ros.org/melodic/Installation/Source "ROS Melodic") installed with the following packages
  - [ros, ros_comm, roslint, geometry, common_msgs, std_srvs, image_common, image_transport_plugins, usb_cam](https://www.ros.org/reps/rep-0131.html#variants "ros, ros_comm, roslint, geometry, common_msgs, std_srvs, image_common, image_transport_plugins, usb_cam")
 - [create_ap](https://github.com/oblique/create_ap "create_ap") to set up device as an access point
 - [RTIMULib2](https://github.com/RTIMULib/RTIMULib2 "RTIMULib2") for the 9260 imu library
 - [rtimulib_ros](https://github.com/romainreignier/rtimulib_ros "rtimulib_ros") to interface with the 250 imu
 - [ros-pwm-pca9685](https://github.com/dheera/ros-pwm-pca9685 "ros-pwm-pca9685") for the servo driver
 - [gpiozero](https://gpiozero.readthedocs.io/en/stable/ "gpiozero") for gpio
 - [ros systemd automatic startup](https://blog.roverrobotics.com/how-to-run-ros-on-startup-bootup/ "ros systemd automatic startup") of roscore and todd.launch (with modifications)
 - windows/samba share to directly access the toddcore ros files in catkin_ws
 - SSH enabled to access todd directly via Putty or other ssh client

##### todd.launch launches the following ros nodes on bootup:
 - a camera node (320x200 @ 5fps)
 - a servo_node (16 channel pwm servo driver @ 50hz, 16bit, raw pwm probably in range of 2800 to 7600)
 - an imu node (using RTIMULib and sensor fusion, 20 updates a second)
 - an ultrasonic sensor node (5 updates a second)
 - a simple temperature node (for the pi)

##### toddcore configuration:
 - local login is pi / rospberry
 - ssid for access point is todd, password is rospberry
 - rosmaster is 192.168.12.1
 - pi directory is shared as todd, user/password is pi/rospberry
 - create_ap starts at boot (sudo systemctl enable create_ap)
 - roscore starts at boot (sudo systemctl enable roscore.service)
 - roslaunch starts "roslaunch todd todd.launch" at boot (sudo systemctl enable roslaunch.service)

##### what's missing / next / on the FIXME list?
 - RTIMUCal seems to ignore keypresses during calibration, so no imu fusion (yet)
 - current todd samba share is readonly
 - configure access point via config file
 - configure samba fileshare via config file
 - configure nodes via config file:
  - camera parameters (resolution, framerate)
  - ultrasonic sensor parameters (updates per second)
  - imu parameters (updates per second)
 - a way to synchronize time between toddcore and the real world
-------
## FAQ:
#### Why?
- Pairing up the Raspberry Pi Zero W with ROS opens up entirely new opportunities for advanced low-cost robotics.  In this case the Raspberry Pi Zero W handles all the low level robot things, and ROS is used as the communications framework.  In this way you're only limited by network bandwidth and the processing power of your laptop or desktop.
- Recent ROS distributions only support ROS on the Raspberry pi 2B or greater (all arm7l devices) and on Ubuntu Mate.  The Raspberry Pi Zero/W is an arm6l device, so you if you want ROS on it you must compile it yourself which is a bit of a pain.  So I put together all of my work in a separate repository so others can take advantage of a prebuilt ROS Melodic setup.
- You can pick up a Pi Zero / W for a little as $5, which opens up all kinds of options for small projects

#### How?
 - I downloaded and configured all of the required sources and compiled them on a Raspberry Pi 4 (I guess you could do it directly on a zero, it just takes *alot* longer). 

#### Will this work on any Raspberry Pi?
 - In theory this should work on any Pi from the zero and original A+ to the 4B.

#### What if I want my project to do something different / launch different nodes / not launch some of those nodes / write my own nodes?
 - toddcore is set up to run the launchfile ~/catkin_ws/src/todd/launch/todd.launch on boot.  toddcore is a fully-compliant ROS installation, so any changes made in catkin_ws and todd.launch will take effect after the reboot (or sudo systemctl restart roslaunch.service)

#### What if I need to connect toddcore back to the internet?
 - no problem, just do the following:
  - sudo systemctl disable create_ap
  - sudo raspi-config (then follow steps to configure wifi network)
  - reboot
  - do your internet thang (hopefully you've already connected a monitor and keyboard)
  - sudo systemctl enable create_ap
  - reboot

#### What if I want to change the machine name / hostname?
 - sudo nano /etc/hostname
 - sudo nano /etc/hosts
 - reboot

#### What if I want to change the login password?
 - log into pi locally or through SSH
 - [passwd](https://www.raspberrypi.org/documentation/linux/usage/users.md "passwd") 

#### What if I want to change the windows / samba fileshare password?
 - [smbpasswd -U pi](https://www.dark-hamster.com/operating-system/linux/ubuntu/reset-samba-user-password-via-command-line/ "smbpasswd -U pi")

#### What if I want to change the accesspoint SSID / password?
 - [sudo create_ap -n wlan0 todd rospberry --mkconfig /etc/create_ap.conf](https://github.com/oblique/create_ap#ap-without-internet-sharing "sudo create_ap -n wlan0 todd rospberry --mkconfig /etc/create_ap.conf")
 - reboot
___
## Requirements:
 - A Raspberry Pi Zero W
 - Some way of connecting to it's access point via wifi
 - (optional, but helpful) mini hdmi adapter and display
 - (optional, but helpful) microusb to usb adapter and keyboard
