# Devops Challenge 4 
Task 1:
*	Launch an EC2 instance in the source region. (us-east-1) N-Virginia
- (Created new ec2 Instance in default VPC, default Subnet, default Security-Group) 
```
aws ec2 run-instances --image-id ami-0866a3c8686eaeeba --count 1 --instance-type t2.micro --key-name challengeKP --security-group-ids sg-0f95664bdcad8f2b1 --subnet-id subnet-0f14a93b170f0e118
```

*	Install software packages (e.g., web server, monitoring tools). 
(From script.txt I have installed Nginx[http:80] and Grafana[tcp:3000] also made it enabled on system restart. Added inbound rules for ports http and 3000. )

```
# Installing Nginx and Grafana to make Custome AMI 
#Update package lists and install nginx
apt-get update
apt-get install -y nginx

# Enable Nginx to start on boot
systemctl enable nginx

#Install the prerequisite packages of Grafana:
sudo apt-get install -y apt-transport-https software-properties-common wget

#Import the GPG key:
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null

#To add a repository for stable releases, run the following command:
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

# Updates the list of available packages
sudo apt-get update

# Installs the latest OSS release of Grafana:
sudo apt-get install grafana

#To start the service, run the following commands:
sudo systemctl daemon-reload
sudo systemctl start grafana-server

#To configure the Grafana server to start at boot, run the following command:
#Grafana will be accessed from <public-ip>:3000 
sudo systemctl enable grafana-server.service
```


*	Bake these changes into a custom AMI. (Create your custom AMI / Golden Image) /n
```
aws ec2 create-image --instance-id i-123456asdb --name "My Golden Image" --description "My challenge 4 custom AMI" 
```

Task 2:
* Copy the custom AMI to a target region. (Copied AMI from North-Virginia us-east-1 to Ohio us-east-2)
```
aws ec2 copy-image --region us-east-2 --name "My Golden Image" --source-region us-east-1 --source-image-id ami-05545hunjn --description "This is my Golden Image copied from N.Virginia"
```

Task 3:
* Launch a New Instance from the Custom AMI
```
aws ec2 run-instances --image-id ami-xxami-of-goldnn --count 1 --instance-type t2.micro --key-name MyKeyPair --security-group-ids sg-903004f8 --subnet-id subnet-6e7f829e
```

Task 4:	Prepare a Volume Snapshot
*	Create a snapshot of an existing volume in the source region. (Created volume in dest region and created snapshot of it and moved to source region and attached to instance there)
*	Copy the snapshot to the target region.

* creating Snapshot of existing volumeA
```
aws ec2 create-snapshot --volume-id vol-xyzabc --description "This is snapshot of root volume"
```

* copy snapshot to target region
```
aws ec2 copy-snapshot --region us-east-2 --source-region us-east-1 --source-snapshot-id snap-066877671789bd71b --description "This is snapshot of root volume coped from us-east-1"
```

Task 5: 
* creating new volume from copied snapshot and attach it to instance
```
aws ec2 create-volume volume-type gp3 --size 10 --snapshot-id snap-066877671789bd71b --availability-zone us-east-1b
```

* creating new volume just for practice 
```
aws ec2 create-volume --volume-type gp3 --size 10 --availability-zone us-east-1b
```

* attach volume to EC2 Instance 
```
aws ec2 attach-volume --volume-id vol-abcxys --instance-id i-abc03454 --device /dev/sdf
```

