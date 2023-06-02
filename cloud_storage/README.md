# bdp2-2022 - Cloud Storage
This repository contains files used in the course <b>Infrastructures for Big Data Processing</b> (BDP2) at the University of Bologna, Academic Year 2022-2023, taught by prof. Davide Salomoni.

For details, see the course slides.

For more information on the course, see <a href=https://www.unibo.it/en/teaching/course-unit-catalogue/course-unit/2021/435337>here</a>.

## Create a directory for this module on VM1 and go there
```
mkdir -p ~/cloud_storage
cd ~/cloud_storage

```

## Installation of the words list
```
sudo apt install -y wamerican

```

## Examples of `wget` and `scp` commands

(change them according to your own set up!)
```
wget https://github.com/dsalomoni/bdp2-2022/raw/main/cloud_storage/taipei.jpg
wget https://github.com/dsalomoni/bdp2-2022/raw/main/cloud_storage/one_image.py
scp -i ~/.ssh/bdp2-2021.pem ubuntu@ec2-3-84-187-177.compute-1.amazonaws.com:/home/ubuntu/cloud_storage/taipei_bw.pdf .
```
__Change the name and location of the PEM file as well as the name of the VM1 machine__ adapting it to your own environemnt.

## Installation of an NFS server on VM2

### Prepare the directory

On VM2, create the directory to host the files:

```
sudo mkdir /data

```

Get the image files, decompressing them into the /data directory:

```
wget https://tinyurl.com/4sdx42bz -O - | sudo tar -zx -C /data

```

Change permissions to allows reading and writing from VM1:

```
sudo chown -R nobody:nogroup /data
sudo chmod -R 777 /data

```

### NFS server installation on VM2

```
sudo apt update && sudo apt -y upgrade
sudo apt install -y nfs-kernel-server

```

Put the following line at the end of the `/etc/exports` file to export the /data directory to VM1 only:

```
/data   <private_VM1_address>(rw,sync,no_subtree_check)
```

### Export the shared directory

```
sudo exportfs -a
sudo systemctl restart nfs-kernel-server

```

## Modify the security group on VM2

In the "Inbound rules", click on "Edit inbound rules", "Add Rule" and then select NFS, putting as source the VM1 private IP address, followed by the string /32. For example, 172.31.22.67/32. 

Click on Save rules and verify that the VM2 security group now allows NFS inbound.

## Mount the NFS directory on VM1

First install the NFS client package:
```
sudo apt update && sudo apt -y upgrade
sudo apt -y install nfs-common
sudo mkdir /remote_data

```

Then mount the remote data directory from VM2:
```
sudo mount <private VM2 address>:/data /remote_data/
```

Check that everything is fine with

```
df -h
ls /remote_data

```

## Process the images

### Images via NFS

Write on VM1 a program called `multiple_images.py` to process the images made available via NFS. Measure the time it takes to process the images.

### Images locally

Copy the images _locally on VM1_ and __modify the `multiple_images.py` program you just wrote__ to process the images from a local directory, rather than from a NFS-exported directory. Measure the time it takes in this case. 

To copy the images to a VM1 local directory:
```
mkdir -p ~/cloud_storage/local_data
wget https://tinyurl.com/4sdx42bz -O - | tar -zx -C ~/cloud_storage/local_data

```

### Simulating network delay

To introduce a 10ms delay to every packet transmitted on the VM2 ethernet interface:
```
sudo tc qdisc add dev eth0 root netem delay 10ms
```

To remove the 10ms delay introduced above:
```
sudo tc qdisc del dev eth0 root netem delay 10ms
```