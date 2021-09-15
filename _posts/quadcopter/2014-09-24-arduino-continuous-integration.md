---
layout: post
title: Arduino Continuous Integration
date: '2014-09-24T23:47:00+08:00'
tags:
- arduino
- jenkins
- headless
- continuous integration
- ci
- ubuntu server
- ubuntu
- virtualbox
---
Arduino projects should not require continuous integration. Most of them define a single sketch with only a few files in one folder: it's easy to verify that the code compiles, and a developer is unlikely to miss files while committing. This said Robokitchen has configurable libraries reused across different sketches. It would be good to check that changes in the libraries don't break any of the sketches which depend on them.  
  
The Linux version of the Arduino IDE 1.5.2+ supports command line compilation. We're going to set up a Jenkins server to compile all the sketches whenever a change is pushed to GitHub. We use VirtualBox and Ubuntu Server 14.04.1 LTS, but any other virtualisation software or distribution should work equally well.  
  
**VM setup**  
  
We keep the default settings for the VM (512Mb RAM, 8Gb HDD) and Ubuntu. We just add OpenSSH server and Tomcat at the end. Once the setup is complete we turn the machine off and add a host-only network adapter. It operates on the 192.168.56.0/24 subnetwork by default in VirtualBox. We restart and add the following lines in /etc/network/interfaces.  
  
auto eth1  
iface eth1 inet static  
address 192.168.56.10  
netmask 255.255.255.0  
  
After restarting we should be able to SSH the machine on 192.168.56.10. After confirming this works we can reboot and run the VM headless, holding shift as we click start.  
  
**Arduino IDE headless**  
  
Let's now download and install the latest Arduino IDE.  
  
wget [https://downloads.arduino.cc/arduino-1.5.7-linux64.tgz](https://downloads.arduino.cc/arduino-1.5.7-linux64.tgz)  
tar -xvf arduino-1.5.7-linux64.tgz  
sudo mv arduino-1.5.7/ /usr/local/  
sudo chown root:root -R /usr/local/arduino-1.5.7/  
  
Let's clone the Robokitchen repository and compile a test sketch.  
  
sudo apt-get install git  
git clone [https://github.com/marcv81/quadcopter.git](https://github.com/marcv81/quadcopter.git)  
/usr/local/arduino-1.5.7/arduino --verify robokitchen/sketches/Quadcopter/Quadcopter.ino  
  
Bad news, it does not work. As explained in [this issue](https://github.com/arduino/Arduino/issues/1981) the Arduino command line interface still requires a X server to work. No problem we can install Xvfb as a stub.  
  
sudo apt-get install xvfb  
sudo apt-get install xfonts-100dpi xfonts-75dpi xfonts-scalable xfonts-cyrillic  
  
We also need to create the Upstart configuration in /etc/init/xvfb.conf.  
  
description "Xvfb X Server"  
start on (net-device-up  
and local-filesystems  
and runlevel [2345])  
stop on runlevel [016]  
exec /usr/bin/Xvfb :99 -screen 0 640x480x8  
  
Ubuntu Server only provides openjdk-7-jre-headless, so we also need to install the regular openjdk-7-jre.  
  
sudo apt-get install openjdk-7-jre  
  
Once we restart we can export DISPLAY=:99 and try to compile again. Appart from an irrelevant Xvfb error it now works.  
  
**Maven**  
  
I quite like the combination of Maven and Jenkins: the modules are displayed nicely. Let's define our [sketches as modules of a parent project](https://github.com/marcv81/quadcopter/commit/e8f60d2aaa77a4623df398ac3cff0ceb89875ca1). For portability we rely on the ARDUINO_HOME environment variable. We can now install Maven and compile all the sketches at once.  
  
sudo apt-get install maven  
cd robokitchen/sketches  
export DISPLAY=:99  
export ARDUINO_HOME=/usr/local/arduino-1.5.7/  
mvn compile  
  
**Jenkins**  
  
We are going to deploy Jenkins in Tomcat. The Arduino builds should be lightweight, but it's a good practice to set the tomcat7 home directory to a partition with enough space.  
  
sudo service tomcat7 stop  
sudo mkdir /home/tomcat7  
sudo chown -R tomcat7:tomcat7 /home/tomcat7/  
sudo usermod -d /home/tomcat7 -m tomcat7  
sudo service tomcat7 start  
  
Let's now download and deploy Jenkins.  
  
wget [https://mirrors.jenkins-ci.org/war-stable/latest/jenkins.war](https://mirrors.jenkins-ci.org/war-stable/latest/jenkins.war)  
sudo chown tomcat7:tomcat7 jenkins.war  
sudo mv jenkins.war /var/lib/tomcat7/webapps/  
  
The Jenkins URL is [https://192.168.56.10:8080/jenkins](https://192.168.56.10:8080/jenkins). Let's first go to Manage Jenkins > Configure System > Global properties and define the following environment variables.

- DISPLAY=:99
- ARDUINO_HOME=/usr/local/arduino-1.5.7/

On the same screen we should define a Maven installation. On Ubuntu Server 14.04.1 LTS the default is Maven 3.0.5 installed in /usr/share/maven/. Now is a good time to install the GitHub plugin and restart Jenkins. We finally have all we need to create a Maven job. The main parameters are as follows.

- Git repository URL: [https://github.com/marcv81/quadcopter.git](https://github.com/marcv81/quadcopter.git)
- Root POM: sketches/pom.xml
- Goals: clean compile

One run this job should lead to a nice screen full of blue balls.

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2014-09-24-arduino-continuous-integration-1.jpg)
{:refdef}
