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

Update Raspbian OS and install requisite and recommended packages:
```
sudo apt-get update
sudo apt-get install build-essential python-dev
sudo apt-get install lsof
sudo apt-get install xrdp
sudo apt-get install libssl-dev

sudo apt-get install cmake
sudo apt-get install sqlite3

sudo reboot
```

Upgrade firmware:
```
sudo apt-get install rpi-update
sudo rpi-update
```

Optionally - install XRDP for remote desktop:
```
sudo apt-get install xrdp
```

Optionally - generate SSH identity for e.g. forked github project:
```
ssh-keygen -t rsa -b 2048
```
Install AWS CLI and AWS IoT device SDK:
```
pip install --user awscli
pip install --user AWSIoTPythonSDK
pip install --user pipenv
```

Add user Python path to the bottom of $HOME/.profile (make sure to exit the shell and login in again to make changes take effect):
```
PATH=$HOME/.local/bin:$PATH ; export PATH
```

Prepare the Raspberry PI with Adafruit Python library for the [DHT sensors](https://github.com/adafruit/Adafruit_Python_DHT/):
```
mkdir -p ~/dev/git
cd  ~/dev/git
wget https://github.com/adafruit/Adafruit_Python_DHT/archive/master.zip
unzip master.zip
cd Adafruit_Python_DHT-master/
sudo python setup.py install
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

[Prepare for AWS GreenGrass](http://docs.aws.amazon.com/greengrass/latest/developerguide/prepare-raspi.html):
```
sudo adduser --system ggc_user
sudo addgroup --system ggc_group

cat <-EOF >>/etc/sysctl.d/98-rpi.conf
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
EOF

sudo reboot

sudo sysctl -a | grep fs
# Should show:
# fs.protected_hardlinks = 1
# fs.protected_symlinks = 1
```

Create and configure a Systems Manager SSM service role (you will need an AWS IAM user with sufficient privileges and credentials in the shell):
```
cd $HOME/dev/git/raspberry-sensor
aws iam create-role --role-name SSMServiceRole --assume-role-policy-document file://policy/SSMService-Trust.json
aws iam attach-role-policy --role-name SSMServiceRole --policy-arn arn:aws:iam::aws:policy/service-role/AmazonEC2RoleforSSM 
```

Create a Managed-Instance Activation:
```
aws ssm create-activation --default-instance-name raspberry --iam-role RunCommandServiceRole --registration-limit 10 --region eu-west-1 --output json
```

Install AWS SSM Agent:
```
mkdir /tmp/ssm
sudo curl https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/debian_arm/amazon-ssm-agent.deb -o /tmp/ssm/amazon-ssm-agent.deb
sudo dpkg -i /tmp/ssm/amazon-ssm-agent.deb
sudo service amazon-ssm-agent stop

```


