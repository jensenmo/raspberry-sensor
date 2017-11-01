# raspberry-sensor

This project provides scripts and templates to set-up Raspberry Pi as a sensor for temperature and humidity. The data is logged in a text file and in AWS.


## Set-up instructions

The following instructions assume that:
* Raspbian 9 (Stretch) has already been installed, e.g. via [NOOBS](https://www.raspberrypi.org/documentation/installation/noobs.md)
* You're logged in as the pi user

Enable ssh and disable desktop at boot: https://www.raspberrypi.org/documentation/remote-access/ssh/
```
sudo raspi-config
```

Change pi password:
```
passwd pi
```

Increase swap to 1GB:
```
sudo sed -i.original 's/^CONF_SWAPSIZE=.*/CONF_SWAPSIZE=1024/g' /etc/dphys-swapfile
sudo dphys-swapfile setup
sudo dphys-swapfile swapon
```

Update Raspbian OS:
```
sudo apt-get update
sudo apt-get install build-essential python-dev
sudo apt install lsof
sudo apt install xrdp
sudo apt install libssl-dev

sudo reboot
```

Upgrade firmware:
```
sudo apt-get install rpi-update
sudo rpi-update
```

Optionally - generate SSH identity for e.g. forked github project:
```
ssh-keygen -t rsa -b 2048
```

Prepare the Raspberry PI with Adafruit Python library for DHT sensor:
```
mkdir -p ~/dev/git
cd  ~/dev/git
wget https://github.com/adafruit/Adafruit_Python_DHT/archive/master.zip
unzip master.zip
cd Adafruit_Python_DHT-master/
sudo python setup.py install
```

Install AWS CLI and AWS IoT device SDK:
```
pip install --user awscli
pip install --user AWSIoTPythonSDK
pip install --user pipenv
```

Add user Python path to the bottom of $HOME/.profile:
```
PATH=$HOME/.local/bin:$PATH ; export PATH
```

Register the thing and download connect_device_package.zip from your local AWS region (eu-west-1 used as example below): https://eu-west-1.console.aws.amazon.com/iotv2/home?region=eu-west-1#/software
```
mkdir -p ~/tmp
cd ~/tmp
unzip ~/Downloads/connect_device_package.zip
chmod ug+x start.sh
sudo /bin/bash
./start.sh
# Go to the AWS Console and see messages flowing through
```

