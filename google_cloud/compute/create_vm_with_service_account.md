This Tutorial shows how to create a virtual machine in google cloud platform with service account. We will be using gcloud cli to 
perform the tutorial

# Prerequisites

1. [Install & Configure gcloud cli](https://cloud.google.com/sdk/docs/install)


## Steps
```
1. Create VPC 
2. Create subnet-1
3. Create subnet-2 with private google access enabled.
4. Create service account
5. Add compute admin role to service account
6. Create virtual machine
7. Create firewall rule to access virtual machine.
8. SSH into virtual machine.
9. Verify virtual machine identity with 'gcloud config list'
```

`
# Login to your google cloud console.
gcloud auth login
`
`
# You can list the active account name with this command
gcloud auth list

PROJECT_ID=<your-project-id>
`
`
# If you want to set the project
gcloud config set project $PROJECT_ID
`
`
# Create a VPC
gcloud compute networks create vpc-1 --project=$PROJECT_ID --subnet-mode=custom --mtu=1460 --bgp-routing-mode=regional
`
`
# Create a subnet in vpc
gcloud compute networks subnets create subnet-1 --project=$PROJECT_ID --range=10.0.1.0/24 --network=vpc-1 --region=europe-west2
`
`
# Create a 2nd subnet in vpc
gcloud compute networks subnets create subnet-2 --project=$PROJECT_ID --range=10.0.2.0/24 --network=vpc-1 --region=europe-west2 --enable-private-ip-google-access
`
`
# Create service account to use it while creating vm
gcloud iam service-accounts create comute-sa --display-name="myproject-compute-sa"
`
https://www.qwiklabs.com/focuses/1038?parent=catalog #If you want to get basic understanding on service accounts

`
# Grant role to the service account you have just created.
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member serviceAccount:comute-sa@$PROJECT_ID.iam.gserviceaccount.com --role roles/compute.admin
`
`
# Create VM and add service account as identity
gcloud compute instances create instance-1 --project=$PROJECT_ID --zone=europe-west2-c --machine-type=e2-micro --network-interface=network-tier=PREMIUM,subnet=subnet-1 --maintenance-policy=MIGRATE --service-account=comute-sa@$PROJECT_ID.iam.gserviceaccount.com --scopes=https://www.googleapis.com/auth/cloud-platform --tags=my-vm --create-disk=auto-delete=yes,boot=yes,device-name=instance-1,image=projects/ubuntu-os-cloud/global/images/ubuntu-2004-focal-v20211212,mode=rw,size=10,type=projects/$PROJECT_ID/zones/europe-west2-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
`
You should now have a virtual machine, but now to connect to the VM you should create a firewall rule. Below command is to create ssh firewall rule to the vm

`
# Firewall rule to allow ssh to the instance(service account)
gcloud compute --project=$PROJECT_ID firewall-rules create public-allow-ssh-fw-rule --direction=INGRESS --priority=1000 --network=vpc-1 --action=ALLOW --rules=tcp:22 --source-ranges=0.0.0.0/0 --target-service-accounts=comute-sa@$PROJECT_ID.iam.gserviceaccount.com
`

# connect to the vm using gcloud command
gcloud beta compute ssh --zone "europe-west2-c" "instance-1"  --project $PROJECT_ID

Above command should take you into the virtual machine. Now use command `gcloud config list` which shows you the service account created part of this tutorial. Which confirms VM runs as service account identity.



