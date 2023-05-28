# bdp2-2023 - Introduction
This repository contains files used in the course <b>Infrastructures for Big Data Processing</b> (BDP2) at the University of Bologna, Academic Year 2022-2023, taught by prof. Davide Salomoni.

For details, see the course slides.

For more information on the course, see <a href="https://virtuale.unibo.it/course/view.php?id=43588">here</a>.

## Accessing the AWS Lab

Follow the instructions provided during the course to get an account and set up your Lab environment. Remember that, once you have set up your account, you should go to <a href=https://www.awsacademy.com/LMS_Login>https://www.awsacademy.com/LMS_Login</a> to log in to your AWS Lab. Select "Student Login" there.

## Configuration of the 2 VMs on AWS

### VM characteristics

- Ubuntu Server 22.04 LTS, 64-bit (x86)
- Size: t2.medium
- Subnet: default in us-east-1c
- Storage: 30 GB volume (**not** the default 8 GB size)
- Security group: rename it and call it "bdp2-security"

Remember to generate a new key pair. Call it e.g. `bdp2`. The private key, called in this case `bdp2.pem`, should be downloaded to your laptop and its access protected from the terminal with the command 

```
chmod 400 bdp2.pem

```

### Software update
```
sudo apt update && sudo apt -y upgrade

```

### Changing the Linux prompt

Put the following at the end of the `.bashrc` file:

```
PS1="\[\033[01;32m\]\u@VM1\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ "
```

The above command is for the VM called `VM1`. Do the same for `VM2`, changing VM1 to VM2 in the `PS1` string above.

Once done, activate the new prompt either by logging out and then back in, or simply typing `source .bashrc`

### Installing additional packages on VM1

```
sudo apt update
sudo apt -y upgrade
sudo apt -y install python3-pip python3-matplotlib
sudo pip install --upgrade numpy Pillow scikit-image

```