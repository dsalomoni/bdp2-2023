# bdp2-2023 - Advanced Containers
This repository contains files used in the course <b>Infrastructures for Big Data Processing</b> (BDP2) at the University of Bologna, Academic Year 2022-2023, taught by prof. Davide Salomoni.

For details, see the course slides.

For more information on the course, see <a href="https://www.unibo.it/it/didattica/insegnamenti/insegnamento/2022/435337">here</a>.

## Create a directory for this module and go there
```
mkdir -p ~/containers
cd ~/containers

```

## Install docker

On both VM1 and VM2 run the following commands:

```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

```

To issue docker commands without `sudo`:

```
sudo usermod -aG docker ${USER}

```

Remember to log out and then log back in to apply this change. Once done, you should be able to issue a docker command such as `docker info` without prefixing it with `sudo`.

## Inspecting bridges

Create a bridge and connect a container to that bridge:

```
docker network create my-bridge
docker run --rm -it --network=my-bridge --name=test1 alpine sh

```
Show the configuration of a bridge:

```
docker network inspect <bridge>
```

Parse the output of this command to print the containers connected to a certain bridge:

```
docker network inspect my-bridge | python3 -c "import sys, json; print([v['Name'] for k,v in json.load(sys.stdin)[0]['Containers'].items()])"

```

## What is my IP address?

Check the real IP address of `test1`, connected to the `my-bridge` bridge:
```
# this is to be done on test1
apk update && apk add bind-tools
dig +short myip.opendns.com @resolver1.opendns.com

```

## Run mypi.py in a container

Create an alpine container, connect to it and run `mypi.py` inside the container:

```
docker run --rm -it --name=test1 alpine sh
# install python3 in the container
apk update && apk add python3 

# write mypi.py in the container
# copy it from this repo

# run mypi.py in the container
python3 mypi.py #
```

Check what's going on in another terminal window on VM1 with `docker top test1`, with `docker stats test1`, as well as with portainer.

## Run an infinite task in a detached container (detached = running in the background)

```
docker run --rm -d --name test1 alpine /bin/sh -c "while true; do $(echo date); sleep 1; done"

```
While the container is running, check its logs with `docker logs --follow test1`. Remember to stop the container when you are done with these checks, with `docker stop test1`.

## Install portainer

```
docker volume create portainer_data
docker run -d --rm -p 8000:8000 -p 443:9443 --name=portainer -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce

```

Remember to change the VM1 security group appropriately or you won't be able to connect to the portainer container. Make sure that as _source_ IP address you put that of your own laptop. Do __not__ open the security group for port 443 to the world.

## Install `git` on VM1

```
sudo apt update && sudo apt install -y git

```

## Configure `git`

```
git config --global user.name "<your name>"
git config --global user.email "<your email>"
```

## Create a local repo

```
mkdir -p ~/containers/myweb
cd ~/containers/myweb
git init

```

## Pushing a new repo to GitHub

Tell git the location of the remote repository (it must be already created on GitHub) and push your changes to GitHub. You will be asked your GitHub username and password. Remember that as password you need to put a GitHub Personal Access Token.

```
git remote add origin https://github.com/dsalomoni/bdp2-test.git
git push -u origin master
```

If you then want to retrieve the latest published version of the files on the repo, use the `pull` command:
```
git pull origin master
```

## Cloning a repo
The command is `git clone`. Try it on VM2.
```
git clone https://github.com/dsalomoni/bdp2-test.git
```

## Using `docker-compose`
On VM1, create a new directory for the docker-compose example and go to that directory:

```
mkdir -p ~/containers/compose
chdir ~/containers/compose
```

Copy the sample `docker-compose.yml` file found in this GitHub repository to the newly created directory. 

By default, the `docker-compose` command is not installed on VM1; install it with:

```
sudo apt upgrade && sudo apt install -y docker-compose
```

Build the application stack with:

```
docker-compose up --build --no-start
```

Start the application stack with:

```
docker-compose start
```

Verify with `docker ps` that the containers are up and running. Open up the page `http://VM1_public_ip:8080/` and you should get the Wordpress set-up page. Take care of the security group, as explained during the course. Notice that if you are on AlmaWifi, port 8080 outgoing from your laptop may be blocked; in that case, change the `port` directive on the compose file to `80:80`.

Stop the application stack with:

```
docker-compose stop
```

Remove it completely (in this case, you will have to build it again the next time you want to run the application stack) with:

```
docker-compose down
```