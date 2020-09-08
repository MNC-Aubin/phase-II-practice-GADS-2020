# Working with Virtual Machines

## Objectives

- In this lab, you learn how to perform the following tasks:
- Customize an application server
- Install and configure necessary software
- Configure network access
- Schedule regular backups

## Task 1: Create the VM

### Create a variable for the project id

```
export YOUR_PROJECT_ID=<Enter your project id here>
```

### Define a VM using advanced options

```

gcloud beta compute --project=$YOUR_PROJECT_ID instances create mc-server --zone=us-central1-a --machine-type=n1-standard-2 --subnet=default --private-network-ip=10.128.0.3 --network-tier=PREMIUM --maintenance-policy=MIGRATE --tags=minecraft-server --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=mc-server --create-disk=mode=rw,size=50,type=projects/$YOUR_PROJECT_ID/zones/us-central1-a/diskTypes/pd-ssd,name=minecraft-disk,device-name=minecraft-disk --reservation-affinity=any

```

## Task 2: Prepare the data disk

### Create a directory and format and mount the disk

1. Connect via SSH

```
gcloud beta compute ssh --zone "us-central1-a" "mc-server" --project $YOUR_PROJECT_ID
```

2. To create a directory that serves as the mount point for the data disk, run the following command:

```
sudo mkdir -p /home/minecraft
```

3. To format the disk, run the following command:

```
sudo mkfs.ext4 -F -E lazy_itable_init=0,\
lazy_journal_init=0,discard \
/dev/disk/by-id/google-minecraft-disk
```

4. To mount the disk, run the following command:

```
sudo mount -o discard,defaults /dev/disk/by-id/google-minecraft-disk /home/minecraft
```

## Task 3: Install and run the application

### Install the Java Runtime Environment (JRE) and the Minecraft server

1. Update the Debian repositories on the VM, run the following command:

```
sudo apt-get update
```

2. Install the headless JRE, run the following command:

```
sudo apt-get install -y default-jre-headless
```

3. To navigate to the directory where the persistent disk is mounted, run the following command:

```
cd /home/minecraft
```

4. To install wget, run the following command:

```
sudo apt-get install wget
```
If prompted to continue, type Y

5. To download the current Minecraft server JAR file (1.11.2 JAR), run the following command:

```
sudo wget https://launcher.mojang.com/v1/objects/d0d0fe2b1dc6ab4c65554cb734270872b72dadd6/server.jar
```

### Initialize the Minecraft server

1. To initialize the Minecraft server, run the following command:

```
sudo java -Xmx1024M -Xms1024M -jar server.jar nogui
```

2. To see the files that were created in the first initialization of the Minecraft server, run the following command:

```
sudo ls -l
```

3. To edit the EULA, run the following command:

```
sudo nano eula.txt
```

4. Change the last line of the file from ```eula=false``` to ```eula=true```

5. Press **Ctrl+O**, **ENTER** to save the file and then press **Ctrl+X** to exit nano.


### Create a virtual terminal screen to start the Minecraft server

1. To install ```screen```, run the following command:

```
sudo apt-get install -y screen
```

2. To start your Minecraft server in a screen virtual terminal, run the following command: (Use the -S flag to name your terminal mcs)

```
sudo screen -S mcs java -Xmx1024M -Xms1024M -jar server.jar nogui
```


### Detach from the screen and close your SSH session

1. To detach the screen terminal, **press Ctrl+A**, **Ctrl+D**. The terminal continues to run in the background. To reattach the terminal, run the following command:

```
sudo screen -r mcs
```

2. If necessary, exit the screen terminal by pressing **Ctrl+A**, **Ctrl+D**.

3. To exit the SSH terminal, run the following command:

```
exit
```

## Task 4: Allow client traffic

```
gcloud compute --project=$YOUR_PROJECT_ID firewall-rules create minecraft-rule --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:25565 --source-ranges=0.0.0.0/0 --target-tags=minecraft-server
```

### Verify server availability

1. Run command to get external IP address (It is the value of **natIP**)

```
gcloud compute instances describe mc-server --zone=us-central1-a
```

2. Use the following website to test your [Minecraft server](https://mcsrvstat.us/)


## Task 5: Schedule regular backups

### Create a Cloud Storage bucket

1. Connect via SSH

```
gcloud beta compute ssh --zone "us-central1-a" "mc-server" --project $YOUR_PROJECT_ID
```

2. Create a globally unique bucket name, and store it in the environment variable YOUR_BUCKET_NAME. To make it unique, you can use your Project ID. Run the following command:

```
export YOUR_BUCKET_NAME=<Enter your bucket name here>
```

3. Verify it with echo:

```
echo $YOUR_BUCKET_NAME
```

4. To create the bucket using the gsutil tool, part of the Cloud SDK, run the following command:

```
gsutil mb gs://$YOUR_BUCKET_NAME-minecraft-backup
```

### Create a backup script

1. In the mc-server SSH terminal, navigate to your home directory:

```
cd /home/minecraft
```

2. To create the script, run the following command:

```
sudo nano /home/minecraft/backup.sh
```

3. Copy and paste the following script into the file:

```
#!/bin/bash
screen -r mcs -X stuff '/save-all\n/save-off\n'
/usr/bin/gsutil cp -R ${BASH_SOURCE%/*}/world gs://${YOUR_BUCKET_NAME}-minecraft-backup/$(date "+%Y%m%d-%H%M%S")-world
screen -r mcs -X stuff '/save-on\n'
```

4. Press **Ctrl+O**, **ENTER** to save the file, and press **Ctrl+X** to exit nano.

5. To make the script executable, run the following command:

```
sudo chmod 755 /home/minecraft/backup.sh
```

### Test the backup script and schedule a cron job

1. In the mc-server SSH terminal, run the backup script:

```
. /home/minecraft/backup.sh
```

2. In the mc-server SSH terminal, open the cron table for editing:

```
sudo crontab -e
```

3. When you are prompted to select an editor, type the number corresponding to **nano**, and press **ENTER**.

4. At the bottom of the cron table, paste the following line:

```
0 */4 * * * /home/minecraft/backup.sh
```

5. Press **Ctrl+O**, **ENTER** to save the cron table, and press **Ctrl+X** to exit nano.


## Task 6: Server maintenance

### Connect via SSH to the server, stop it and shut down the VM

1. In the mc-server SSH terminal, run the following command:

```
sudo screen -r -X stuff '/stop\n'
```

2. Stop server

```
gcloud compute instances stop mc-server --zone=us-central1-a
```


### Automate server maintenance with startup and shutdown scripts

Run the following command to add **Custom metadata**:

```
gcloud compute instances add-metadata mc-server --zone us-central1-a  --metadata=startup-script-url=https://storage.googleapis.com/cloud
-training/archinfra/mcserver/startup.sh
```

```
gcloud compute instances add-metadata mc-server --zone us-central1-a  --metadata=shutdown-script-url=https://storage.googleapis.com/clou
d-training/archinfra/mcserver/shutdown.sh
```