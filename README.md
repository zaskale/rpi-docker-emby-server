# rpi-docker-emby-server

The following guide will enable you to run an Emby server as a Docker container on your Raspberry Pi with an attached storage device.

## **Mount your External Storage Device**

Ensure that your Raspberry Pi is up to date by running:

`sudo apt update`

`sudo apt upgrade`

Connect external SSD and check if it's dedected by OS:

`sudo fdisk -l`

Create mount point:

`mkdir /media/pi`

Mount SSD partition to directory:

`mount /dev/sda1 /media/pi`

Check if SSD partition is mounted correctly:

`mount | grep sda1`

Look up the SSD's UUID:

`sudo blkid`

Once you get the UUID output, copy the device UUID and add it to the fstab config file:

`sudo nano /etc/fstab`

Add the following line of code to the bottom of the fstab config file and save it. This will enable your SSD to be mounted on boot:

`UUID=[your uuid] /media/pi [drive type] defaults,nofail 0 2`

## **Install Docker and Portainer**

Run this script to install Docker on your Raspberry Pi:

`curl -sSL https://get.docker.com | sh`

After the script completes, you need to give your Pi user account access to Docker:

`sudo usermod -aG docker $USER`

After the user has been added, you will run a command to download the latest Portainer image for the ARM processor:

For 32 bit OS - `sudo docker pull portainer/portainer-ce:linux-arm`

For 64 bit OS - `sudo docker pull portainer/portainer-ce:linux-arm64`

After the scripc completes, create a new container that will run Portainer. If you are already using port 9000 on your Raspberry Pi for something else, you will need to change the ports below:

For 32 bit OS - `sudo docker run --restart always -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:linux-arm`

For 64 bit OS - `sudo docker run --restart always -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:linux-arm64`

You should now be able to navigate to the IP address of your Raspberry Pi and port 9000 to access Portainer. Open up your web browser, and enter the following address. Once opened, create a username and password:

`http://[RASPBERRY_PI_IP_ADDRESS]:9000`

Select **Local** and click _Connect_. Your Portainer should how be up and running.

## **Install Emby**

In the following steps, you will install Emby in your Portainer. 

In your Portainer Dashboard, click on _local_ under **Environments**, select **Volumes**, and click on _add volume_.

Give the volume a name (e.g. Emby), and click on _Create the volume_. Take note of the Mount Point.

Next, select **Stacks**, and click on _add stack_.

Give the container a name (e.g. emby-server) and paste the stack below. Be sure to update the _time zone_ and the local path to your _config folder_ and _media folder_ that you have set up in the previous steps.

```
version: "2.1"
services:
  emby:
    image: ghcr.io/linuxserver/emby
    container_name: emby
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Detroit
    volumes:
      - /var/lib/docker/volumes/Emby/_data:/config
      - /media/pi:/Media
    ports:
      - 8096:8096
      - 8920:8920 
    restart: unless-stopped
```

Once completed, click on _Deplay the Stack_. When it’s finished, you can access it by navigating to the IP address of your Raspberry Pi and port 8096.

`http://[RASPBERRY_PI_IP]:8096`

## **Emby Setup**

Select your **Preferred Language**.

Create **username** and **password**.

Select your **Content Type**, then add the folders that you mapped earlier. You will need to do this for each content type.

Select your **Metadata** language.

Configure **Remote Access** if you'd like to access your content from outside of your network.

Accept the **Terms of Service**, and click _Finish_. You will be able to login using the username and password, and your media library will start to sync.
