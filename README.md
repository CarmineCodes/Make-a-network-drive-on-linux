# How to create a network drive on linux

This writeup will go over how to create a network drive on linux as a budget nas using docker with a samba container.

How to install Docker and Portainer: https://www.youtube.com/watch?v=GYWsPWtlYXA&t=3s&ab_channel=BarmineTech

Docker info can found here: https://github.com/novaspirit/pi-hosted
The docker info above can be used on ARM or AMD64 boards



## Getting Started

To start, add a clean drive to the machine you would like to run the network share off of. The drive cannot have any partitions on it if you are working with proxmox as it will not see it then. After the drive is connected, console into the machine and verify you can see it listed in the drives with lsblk. If you do not see your new drive, try wiping it again and try these steps again. If it is a new drive and it does not show, check your cable and ports are good and verify it is not a DOA drive.

## Partioning And Assinging File Structure

The process to set this up is simeple, just verify that you select the right drive.

First, locate the drive you plan to use for your network share by running the command:

    lsblk

This should so all drive seen by the machine, your new drive will have no partitions on it so it should look someting like this
![image](https://user-images.githubusercontent.com/63487881/205470589-06f419c4-e3de-4998-9b2d-94f1521b88be.png)

A you will see other drives listed with partitions and they will look like this.

![image](https://user-images.githubusercontent.com/63487881/205470605-cd1f608a-4e77-4084-808c-58ba957ed9a0.png)

The drive with no partitions is the drive we will be working with.


Now we will use fdisk to partition the drive, ensure you select the proper drive as this will get rid of any data on a drive if you select the wrong one.

    sudo fdisk /dev/yourdisk

Now add a new partition with n

Then make the partition primary with p

Make 1 partition and use all space

When fdisk says it is done enter w to write it to the table


Now the drive is partitioned and ready to be assigned a file structure.

Use 
    
    sudo mkfs.ext4 /dev/yourdrive

To make your new drive use the ext4 file structure.

Your new drive should look similar to this when you run lsblk again

![image](https://user-images.githubusercontent.com/63487881/205470930-19684b0b-eb32-4563-897f-2f548475fe70.png)


## Make the directory for your network share

We need to make an actual directory that our network share will be based out of. It will be accessible on the local machine but also from anywhere on the local network after we finish setting it up.

In the console into the machine you are working on, run the following command

    sudo mkdir nas
 
You can replace nas with whatever you would like to call your share.

## Mounting the drive to its directory

Now the drive needs to be mounted to the directory. That can be done with the following commands.

    sudo mount /dev/yourdrive /nas
    
You will replace /yourdrive with whatever your drive is named and /nas with whatever you named your share from the step above.

## Set Drive To Automount

Your new share is currently mounted but if the machine turns off it may be unmounted and your machine will most likely not be able to remount it. With the following steps we will make it so on startup, the drive is always automounted and you dont lose your new nas.

Locate which drive you are working with by running:

    lsblk

When you see which drive you are working with, run the following command to find its uuid

    blkid
    
The output should be something like this.

![image](https://user-images.githubusercontent.com/63487881/205471305-ae76b8d2-8edf-4003-912c-4e1e50639d07.png)

Find your drive your working with and get its uuid, maybe paste it into a notepad so you dont lose it for the next step.

Now run 

    sudo nano /etc/fstab
    
the /etc/fstab is a file that just makes mounting much easier and helps the system automate it.

After you open the file, itll look something like this:

![image](https://user-images.githubusercontent.com/63487881/205471689-20ef3076-d029-472a-9132-ed72b98228c1.png)

In the file, go down to a new line and enter the following text

    uuid=c21818a0-e0fe-42a3-b9d8-12f89fc94092       /nas    ext4    defaults        0       0

Formatting is important for this so follow it like this:

replace the uuid with the one you got earlier hit tab, enter /yourshare tab, ext4, tab, defaults, tab, 0 tab 0

It should similar to this in your word processor

![image](https://user-images.githubusercontent.com/63487881/205471916-75a83227-5336-4686-9b13-417db75e56f6.png)


## Setup the Samba Docker

Now that everything is setup on the linux side for the new network share, so open up the portainer web portal and if you havent already installed the samba docker container, find it in the app template and install it.

![image](https://user-images.githubusercontent.com/63487881/205472273-501f80c7-c902-43e3-b350-f2439f7f6f12.png)

After the container is installed, click on it and find the duplicate and edit button

![image](https://user-images.githubusercontent.com/63487881/205472334-5c022a8f-cd87-44b3-9055-ac7c2c76e4fb.png)

From there, scroll down until you see the volumes tab and click on it

![image](https://user-images.githubusercontent.com/63487881/205472351-1514480b-e715-4845-8438-7c6f19a17143.png)

In here change the host option (the circles one) for the /share to what you set your network share to in linux.

![image](https://user-images.githubusercontent.com/63487881/205472376-a28fb5d4-8ccd-4540-925d-9becf013b0c5.png)


This will make it that docker knows where to look for your network share and can pass it to be used.

If youd like to change the default login credentials to your new nas, do the following.

Navigate to the Env tab

![image](https://user-images.githubusercontent.com/63487881/205472466-aeaad148-378b-4a50-8164-8df0db983b29.png)

Then find the name row

![image](https://user-images.githubusercontent.com/63487881/205472480-d5032b14-cc16-4768-8aa8-0979455f9c36.png)

here you can change it to whatever username and password youd like. keep the formatting and just change the text.

After you finished this, everything is all configured, scroll back to the top and click on deploy container.

![image](https://user-images.githubusercontent.com/63487881/205472505-a97f3e45-2495-409e-92dd-c2a40726495f.png)

Now your container will restart and when it turns green its ready to be accessed.

## Connecting to your new nas

On a machine connected to the same network as your new nas, open a file explorer and type in 

    //nasip/nas
    
replace nasip with the ip address of the machine you set your new nas up on and replace nas with whatever you named your share from earlier.

itll prompt you to enter credentials and enter whatever credentials you gave it.

You test you are able to write a file to the new drive and if you can your all set, if not, read through the writeup again and make sure you didnt miss a step.


