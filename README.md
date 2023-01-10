# Install NginX load balancer-provisioned-using Teraform
Provisioned and configured Nginx web server through automation
Prerequisites:
-Installed Terraform
-AWS Account
-Create access & Secret Access key in your AWS account.
-https://registry.terraform.io/providers/hashicorp/aws/latest/docs

Create main.tf file for configuration
Create VPC , Internet_gateway and subnet.

VPC with CIDR 10.0.0.0/16 following name label= “aws_vpc”
Internet gateway with name label “aws_internet_gateway” & attribute as “igw” and we want to associate our internet gateway with “aws_vpc” what we have just created .
Though, inside configuration block of “igw” we we’ll provide a single argument vpc_id= aws_vpc.vpc.id as refrence syntax.
Now, you might me wondering what arguments and attributes are available for a source.

Well to get into you can refer to this https://registry.terraform.io/providers/hashicorp/aws/latest/docs document for AWS.
1 Public Subnet with CIDR 10.0.0.0/24 (us-east-1a) , and we will attach subnet with our VPC. 
And we’re setting up map_public_ip_on_launch=true , so when we spin up an EC2 instance in this subnet it’ll get a Public IP address.

Create Route table & associate it with subnet
We’ll create “aws_route_table” called as “rtb”
Route Table with name label “aws_route_table” and we’ll to associate it to our “aws_vpc”.
Then we’ll create a nested block inside route table resource. Here we’ll be providing a default route 0.0.0.0/0 and putting it 
in our internet gateway through which our traffic can go out of our VPC.
At last we’ll create “aws_route_table_association” called “rta-subnet1” i.e= “aws_route_table_association” “rta-subnet1”.
within this configuration we’ll provide subnet_id and route_table_id so now there is association between those two resources.

Create Security Group || INBOUND & OUTBOUND Rules

Moving further we’ll create an “aws_security_group”.
That allows port 80 from anywhere to talk our EC2 instance.
We’ll be associate security group to our VPC “aws_vpc.vpc.id”
Then , for inbound rule, we’ll setup a single “ingress” group to allow incoming traffic from anywhere.
Which will be defined as from_port =80, to_port =80, protocol =”tcp” and the cidr_blocks= [“0.0.0.0/0”] 
which allows incoming traffic to our intances from anywhere
Lastly we’ll define “egress” group which AWS by default allows all outgoing traffic from an EC2 Instance.

Deploy your first infrastructure

IMP:: Before going with deployment, make sure you have added you Access & Secret_Access key to the configuration file.

Go ahead save the file once you saved your access keys.
Now open your Terminal, cd into the project directory.

$ cd web-server 
$ sudo terraform init
$ sudo terraform plan -out d3.tfplan

We have 7 Resources to get crated to deploy our first web-server with Terraform. 
Let’s go and apply our first plan saved in the file d3.tfplan.
sudo terraform apply "d3.tfplan"

It will start creating required resources, this may take while.

Best and most fascinating about terraform is we can take down all the infrastructure with just a single command line.

terraform destroy
