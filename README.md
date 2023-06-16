# bdp2-2023 - Introduction
This repository contains files used in the course <b>Infrastructures for Big Data Processing</b> (BDP2) at the University of Bologna, Academic Year 2022-2023, taught by prof. Davide Salomoni.

For details, see the course slides.

For more information on the course, see <a href="https://www.unibo.it/it/didattica/insegnamenti/insegnamento/2022/435337">here</a>.

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

**Important note for Windows users**: if your laptop runs Windows, in order to connect to the AWS VMs you may:
  - Use an ssh client, such as MobaXterm (look it up on the Internet to find download and usage instructions).
  - Use the Windows Powershell and the Windows Linux Subsystem (WSL). However, in this case remember to <u>move the `bdp2.pem` private key to your WSL home directory</u>. With the Windows File Explorer, you may go to `MyPC` and there you should find the WSL Linux home folder, called something like `Ubuntu`; inside that folder there will be a `home` folder, and inside that folder there will be a folder with your Windows username. That will be your WSL home directory. Once you have moved the `.pem` file there, remember to issue the `chmod` command shown above. 

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

Perform these commands on VM1:

```
sudo apt update
sudo apt -y upgrade
sudo apt -y install python3-pip python3-matplotlib
sudo pip install --upgrade numpy Pillow scikit-image

```

[![DOI](https://zenodo.org/badge/646546452.svg)](https://zenodo.org/badge/latestdoi/646546452)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
