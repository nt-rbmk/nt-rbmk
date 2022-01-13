**INFO**

In this repo you can find step-by-step insructions to create a Log aggregation/Monitoring mechanism against any running container.
Terraform will be used to create a basic infrastructure on AWS, consisting of VPC, subnet(s), routes, security groups and finally an EC2 that will contain a dockerized version of Grafana, Prometheus, Promtail, Loki and Cadvisor.

**Technologies to be used**
- AWS
- Terraform
- Ansible
- docker

**What you need**
- A linux distro (preferably Ubuntu 20.04)
- An AWS account
- terraform v1.1.3
- ansible [core 2.12.1]
  with the following collections:
  
  ansible.posix     1.3.0  
  community.docker  2.0.1  
  community.general 3.6.0  
- python 3.8.10

**Let' s start**
1. After creating the AWS account it' s necessary to create the access keys:
   On AWS console navigate to IAM -> Users -> Add Users (don' t forget to attach the AdministratorAccess policy) and then under Security Credentials tab create an access key.
2. In the linux shell you need the following for the access key (feel free to name the profile as you like):
```
   :~# cd ~
   :~# mkdir .aws
   :~# cat .aws/config 
   [profile nt-rbmk]
   region = eu-central-1
   output = json
   :~# cat .aws/credentials
   [nt-rbmk] 
   aws_access_key_id = AKXXXXXRU
   aws_secret_access_key = 00XXXXXXXXBU
```
3. Checkout the code from this repo and let' s prepare the terraform environemt, by creating the S3 bucket that will store the terraform state.
   
   **NOTE**: Please use your own profile name and a different S3 bucket name.
```
   :~# ls
   :~# _variables.tf  ansible  main.tf  modules  pre
   :~# cd pre
   :~# terraform init
   :~# terraform apply
```
   
4. Create the infrastructure + ec2:

   **NOTES**: Please change _myip_ with your own public IP address (line 5) at modules/vpc/_variables.tf.
              Please change  _keyname_ (line 15), _private_key_file_ (line 38) and _private_key_ directory (line 44) at modules/ec2s/demo-ec2.tf. Keep in mind that you should change /etc/ansible/hosts file and add the elastic_ip that will be attached to the ec2 during its creation. Otherwise ansible execution will fail.
```
   :~# cd ..
   :~# ls
   :~# _variables.tf  ansible  main.tf  modules  pre
   :~# terraform init
   :~# terraform apply
```
5. Configure Grafana:

   Type http://**ec2_public_ip**:3000 into your web browser and login using _admin_:_admin_
   
   Then click Configuration -> Data sources -> Add data source and select Prometheus. Add in URL: http://prometheus:9090 -> Save & test.
   
   Then select Loki and add in URL http://loki:3100
   
   Finally, click Create -> Import add 893 and Load. Select Prometheus Data source and import. The dashboard for docker containers stats is ready.



**Improvements/TBD**

Create a separate security group for EC2s in public subnet(s)

Create and attach an EIP to the EC2 (In that way, we know in advance the EC2's public IP so we can add it into ansible/hosts file)

Create a site2site VPN and add the EC2 into the private subnet.
