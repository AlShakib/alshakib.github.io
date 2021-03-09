+++
toc = true
date = 2020-09-09T16:42:25+06:00
title = "Create a Custom Image of Fedora Cloud for Google Compute Engine"
description = "Fedora has been my primary operating system for many years. Recently I was doing some computation in the Google Compute Engine. I’ve been disappointed that there is no prebuilt image of Fedora in Google Compute Engine. But They provide a way to create custom images from source disks, images, snapshots, or images stored in Cloud Storage. We can use these images to create virtual machine (VM) instances. In this article we use Fedora Cloud Base Image to create the custom image for Google Compute Engine virtual machine instances."
featured_image = "/images/fedora_cloud_32_for_gcp/featured_image.jpg"
tags = ["Fedora", "Google Compute Engine", "Tutorial"]
categories = ["Linux"]
keywords = ["fedora", "cloud", "fedora 32", "linux", "google compute engine", "google cloud", "google cloud platform", "gcp", "how to", "how to create custom image for google compute engine", "how to create fedora custom image for google compute engine"]
slug = "create-a-custom-image-of-fedora-cloud-for-gcp"

+++

## Introduction

Fedora has been my primary operating system for many years. Recently I was doing some computation in the Google Compute Engine. In case you don’t know about Google Compute Engine, you can start from [here](https://cloud.google.com/compute). I’ve been disappointed that there is no prebuilt image of Fedora in Google Compute Engine. But they provide a way to create custom images from source disks, images, snapshots, or images stored in Cloud Storage. We can use these images to create virtual machine (VM) instances. In this article we use Fedora Cloud Base Image to create the custom image for Google Compute Engine virtual machine instances.

## Preparing environment

To start, you need a linux machine where we’re going to build and modify the official Fedora Cloud image for Google Compute Engine. You also need the gcloud command-line tool to upload and create the image from Google Cloud Storage Bucket. If you don’t have the gcloud command-line tool installed in your machine, you can install it from [here](https://cloud.google.com/sdk/docs/quickstarts).

After installing the gcloud command-line tool, use the `gcloud init` command to perform several common SDK setup tasks. These include authorizing the SDK tools to access Google Cloud using your user account credentials and setting up the default SDK configuration.

## Getting started

To create a custom image for Google Compute Engine, the first thing we will need is a Fedora Cloud Base Image. You can find a full list of Fedora Cloud Images [here](https://alt.fedoraproject.org/cloud). Among those, we download the Cloud Base compressed raw image. In this article, we use Fedora 32 Cloud Base Compressed Raw Image. To download the raw image, run,
```bash
wget "https://download.fedoraproject.org/pub/fedora/linux/releases/32/Cloud/x86_64/images/Fedora-Cloud-Base-32-1.6.x86_64.raw.xz"
```
Once download is done, decompress the downloaded file.
```bash
xz --decompress "Fedora-Cloud-Base-32-1.6.x86_64.raw.xz"
```
This may take a few minutes to decompress. On the next step, we will mount this raw disk image, chroot inside it, install `google-compute-engine-tools` so it can be compatible with Google Compute Engine, remove a few unnecessary software, clean up various temporary files and then finally create a compressed disk image to upload on Google Cloud Storage.

## Customizing the raw image

First create an empty directory, where we can mount the raw disk image.

```bash
mkdir "cloud"
```
And then mount the raw disk image to the `cloud` directory,
```bash
sudo mount -o loop,offset=1048576 "$PWD/Fedora-Cloud-Base-32-1.6.x86_64.raw" "$PWD/cloud"
```
Change the current directory to the `cloud` directory and mount a few more directories  for any decent application and network to work.
```bash
cd "cloud"
sudo mount --bind "/dev" "dev" && sudo mount --bind "/sys" "sys" && sudo mount --bind "/proc" "proc" && sudo mount --bind "/etc/resolv.conf" "etc/resolv.conf"
```
Now, we can chroot to the mounted raw disk
```bash
sudo chroot "./" "/usr/bin/bash"
```
You have successfully chroot to Fedora loop-mounted raw disk.
Let’s update the loop mounted system.
```bash
dnf upgrade --assumeyes --nogpgcheck "*"
```
It may take a few minutes to complete. Once done, install `google-compute-engine-tools` and `cloud-utils-growpart` to make this image compatible with Google Compute Engine and  resize the disk using `cloud-init`.
```bash
dnf install --assumeyes --nogpgcheck "google-compute-engine-tools" "cloud-utils-growpart"
```
This is going to install all Google Cloud related all softwares. Once installation is done, let’s enable a few systemd services to run on boot.
```bash
systemctl enable "google-accounts-daemon" "google-clock-skew-daemon" \
    "google-instance-setup" "google-network-daemon" \
    "google-shutdown-scripts" "google-startup-scripts"
```
Since we installed and removed a few softwares, let’s relabel the entire file system on the next boot. We don’t want selinux to give us any headache.
```bash
touch "/.autorelabel"
```
Now clean dnf caches, clear bash history, exit chroot and get out of the loop-mounted directory.
```bash
dnf clean all
cat /dev/null > /root/.bash_history && history -c && exit
cd ../
```
After getting out of the loop-mounted directory, you can unmount the directories.
```bash
sudo umount "cloud/dev" "cloud/proc" "cloud/sys" "cloud/etc/resolv.conf"
```
After that, remove temporary files and unmount the `cloud` directory.
```bash
sudo fstrim --verbose "cloud"
sudo umount "cloud"
```
Now it’s time to create the final image.
```bash
mv "Fedora-Cloud-Base-32-1.6.x86_64.raw" "disk.raw"
tar --create --auto-compress --file="Fedora-Cloud-Base-32-1.6.x86_64.tar.gz" --sparse "disk.raw"
```
You have successfully created a custom compressed image that we are going to upload in the next step to the Google Cloud Bucket.

## Creating the custom image for Google Compute Engine

Till now, we created a custom compressed image from Fedora Cloud Base raw image. Now we are going to create a Google Cloud Storage bucket and upload this compressed image to that storage bucket.
```bash
gsutil mb "gs://dev-al-shakib-custom-images"
```
This will create a Google Cloud Storage bucket as `dev-al-shakib-custom-images`. Please note that, you can pick only a globally unique name. So choose wisely. Once done, let’s upload our custom image to this bucket.
```bash
gsutil cp "Fedora-Cloud-Base-32-1.6.x86_64.tar.gz" "gs://dev-al-shakib-custom-images/"
```
This may take a few minutes. Once done, create a custom Google Compute Engine image from this uploaded compressed image.
```bash
gcloud compute images create --source-uri \
    gs://dev-al-shakib-custom-images/Fedora-Cloud-Base-32-1.6.x86_64.tar.gz \
    fedora-cloud-base-32
```
Congratulations! You have successfully created the Fedora Cloud Base 32 custom image.

## Testing

Now it’s time to test that custom image.
1. Visit [here](https://console.cloud.google.com/compute/instances)
2. Click on `Create Instance` button
3. Under the `Boot disk` section, click on the `Change` button.
4. Go in the `Custom images` tab and then pick the image you just created.
5. Change the boot disk size to 10GB. 
6. You might want to set the VM as Preemptible, since you’ll use it only for tests and not keep it.
7. Click on the `Create` button.

It may take a few minutes to boot up the instance. Once it’s up and running, click on the `SSH` button. Finally you’re logged in to your Fedora VM, run by Google Cloud Platform.

## Final words

Thanks for reading this article. If you have any question or confusion regarding the tutorial, feel free to ask your questions on [Telegram](https://t.me/AlShakib) or [Twitter](https://twitter.com/_alshakib). You can send me an email as well.
