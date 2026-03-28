---
layout: post
title:  "Create custom OS image on Azure for starting Debian buster VM"
---

# How to Create a custom OS image on Azure for starting Debian buster VMs

Debian 10, nicknamed as `buster` has reached its end of life. If you need to start a new VM using this OS to run your legacy app, these instructions might help.

## Requisite tools
1. az cli (https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt)
2. azcopy cli (https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azcopy-v10#download-azcopy)
3. `wget`, `tar`, `qemu-img`, `stat`


## Process
1. Download the disk image from debian archives
   ```shell
   wget https://cloud.debian.org/images/cloud/buster/latest/debian-10-azure-amd64.tar.xz
   ```

1. Extract the archive to get `disk.raw` file
   ```shell
   tar -xf debian-10-azure-amd64.tar.xz
   ```

2. Convert the disk to VHD format that is compatible with Azure disks
   
   1. Get a multiple of 1 MB that is large enough to hold raw disk
      ```shell
      MB=$((1024 * 1024))
      RAW_SIZE=$(stat -c%s "disk.raw")
      ROUNDED_SIZE=$((($RAW_SIZE/$MB + 1)*$MB))
      ```
   
   1. Ensure the size of the `disk.raw` is a multiple of 1 MB (required by Azure)
      ```shell
      qemu-img resize -f raw disk.raw $ROUNDED_SIZE
      ```

   1. Convert `disk.raw` to fixed-size `debian-buster.vhd`
      ```shell
      qemu-img convert -f raw -o subformat=fixed,force_size -O vpc disk.raw debian-buster.vhd
      ```

1. Choose a resource group and location where you want to work in Azure
   ```shell
   export CHOSEN_GROUP=chosen-group-name
   export CHOSEN_LOCATION=chosen-location-eg-centralus
   ```

1. Upload the VHD contents to an Azure disk
   1. Create disk container
      ```shell
      az disk create \
         --resource-group $CHOSEN_GROUP \
         --name debian-buster-os-disk \
         --location $CHOSEN_LOCATION \
         --upload-type Upload \
         --upload-size-bytes $ROUNDED_SIZE \
         --sku Standard_LRS
      ```
   1. Get SAS write url for the disk container
      ```shell
      SAS_URL=$( \
         az disk grant-access \
            --resource-group $CHOSEN_GROUP \
            --name debian-buster-os-disk \
            --access-level Write \
            --duration-in-seconds 3600)
      ```
   1. Upload `debian-buster.vhd` content to the disk container
      ```shell
      azcopy copy --blob-type PageBlob "debian-buster.vhd" "${SAS_URL}"
      ```
   
   1. Close/conclude writing to the disk
      ```shell
      az disk revoke-access --resource-group $CHOSEN_GROUP --name debian-buster-os-disk
      ```
1. Create an OS image from the uploaded OS disk
   ```shell
   az image create \
      --resource-group $CHOSEN_GROUP \
      --location $CHOSEN_LOCATION \
      --name DebianBusterCustomOSImage \
      --source debian-buster-os-disk \
      --os-type Linux
   ```

1. Once image is generated and visible in Azure portal, the debian-buster-os-disk is no more needed. Delete it to save storage bills;
   ```shell
   az disk delete --resource-group $CHOSEN_GROUP --name debian-buster-os-disk
   ```
   
1. Check if the newly created image with name `DebianBusterCustomOSImage` visible in the azure portal? If yes, use it with the portal to launch VMs just like any other OS for the vm.


## Making `apt update` to work on the Debian buster VM 
Once VM is started and you are able to login to it, if you try `apt update` command, it fails. This is because it trying to pull packages lists from locations that are no more there.
To fix this, edit `/etc/apt/sources.list` file and make sure that all source urls are pointing towards `archive.debian.org`. Once done, save the file and enjoy apt updates as normal.