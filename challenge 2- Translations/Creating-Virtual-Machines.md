# Creating Virtual Machines

## Objectives

- In this lab, you explore the available options for VMs and see the differences between locations.
- In this lab, you learn how to perform the following tasks:
- Create several standard VMs
- Create advanced VMs

## Task 1: Create a utility virtual machine

### Create a VM

### Create a variable for the project id

```
export YOUR_PROJECT_ID=<Enter your project id here>
```

```

gcloud beta compute --project=$YOUR_PROJECT_ID instances create instance-1 --zone=us-central1-c --machine-type=n1-standard-1 --subnet=default --no-address --maintenance-policy=MIGRATE --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=instance-1 --reservation-affinity=any

```

### Explore the VM details

```
gcloud compute instances describe instance-1 --zone us-central1-c
```

### Explore the VM logs

```
gcloud logging read "resource.type=gce_instance" --project=$YOUR_PROJECT_ID
```

## Task 2: Create a Windows virtual machine

### Create a VM

```

gcloud beta compute --project=$YOUR_PROJECT_ID instances create instance-2 --zone=europe-west2-a --machine-type=n1-standard-2 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --tags=http-server,https-server --image=windows-server-2016-dc-core-v20200813 --image-project=windows-cloud --boot-disk-size=100GB --boot-disk-type=pd-ssd --boot-disk-device-name=instance-2 --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any

```

```

gcloud compute --project=$YOUR_PROJECT_ID firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server

```

```

gcloud compute --project=$YOUR_PROJECT_ID firewall-rules create default-allow-https --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:443 --source-ranges=0.0.0.0/0 --target-tags=https-server

```

### Set the password for the VM

```
gcloud beta compute --project $YOUR_PROJECT_ID reset-windows-password "instance-2" --zone "europe-west2-a"
```

Copy the provided password.

## Task 3: Create a custom virtual machine

### Create a VM

```

gcloud beta compute --project=$YOUR_PROJECT_ID instances create instance-4 --zone=us-west1-b --machine-type=custom-6-32768 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=instance-4 --reservation-affinity=any

```

## Connect via SSH to your custom VM


1. Connect via SSH:

```
gcloud beta compute ssh --zone "us-west1-b" "instance-3" --project $YOUR_PROJECT_ID
```

2. To see information about unused and used memory and swap space on your custom VM, run the following command:

```
free
```

3. To see details about the RAM installed on your VM, run the following command:

```
sudo dmidecode -t 17
```

4. To verify the number of processors, run the following command:

```
nproc
```

5. To see details about the CPUs installed on your VM, run the following command:

```
lscpu
```

6. To exit the SSH terminal, run the following command:

```
exit
```
