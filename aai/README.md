# bdp2-2023 - AAI
This repository contains files used in the course <b>Infrastructures for Big Data Processing</b> (BDP2) at the University of Bologna, Academic Year 2022-2023, taught by prof. Davide Salomoni.

For details, see the course slides.

For more information on the course, see <a href="https://www.unibo.it/it/didattica/insegnamenti/insegnamento/2022/435337">here</a>.

## Create a directory for this module and go there
```
mkdir -p ~/aai
cd ~/aai

```
## Self-register your application to IAM

Follow the slides for details on which IAM server to use and how to authenticate to it. A key point is that in the `Redirect URI(s)` field you should put 

```
http://<VM1_public_IP_address>:80/redirect_uri
```

## Building and running the OIDC-enabled container

Edit the file `default.conf` as specified in the slides and build the container image with

```
docker build -t web_server_oidc .
```

Once the build is successful (check that your image has been built with `docker images`) run the container with

```
docker run --rm --name oidc_test -d -p 80:80 web_server_oidc
```

If you now open the URL `http://<VM1_public_IP_address>:80` you should be redirected to the IAM server; log in there, and you should then be automatically redirected to a page on the web server running on VM1.
