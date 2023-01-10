Terraform: Deploy Nginx Server on Public Subnet Modules (Iac)
=============================================================

**What is Terraform?**
======================

**Terrraform** is an open-source Infrastructure as Code (IaC) tool developed by HashiCorp. It is used to define and provision a complete infrastructure using a declarative language in this case we will use a json like (Hashicorp Configuration Language- HCL).IaC helps businesses automate their infrastructures by programmatically managing an entire technology unlike using console service. So, the time is to deep dive into it.

For installation it’s quite simple and straight, please visit [https://www.terraform.io/downloads](https://www.terraform.io/downloads)

Before, we look at the configuration there are three Terraform Object type you need to know about….
**Providers:** Providers block the information of the providers you want to use, in this case we will use AWS

**Resources:** Resources are entity you want to create in a target environment. Each resources are 
associate to a provider. It could be an EC2 INstance, VPC or even a Database. It requires Account information as well as region where you want to set up the target environment.

**Data Sources:** are way to query information from a provider. Just like resources, data sources are also associated with the provider.

**Terraform Workflow:**
=======================

Terraform has a principle workflow, which allows you to Provision, Update and remove infrastructure. Terraform makes use of providers plugins to interact with providers such as AWS, this work at initialization and the command you can use is…

```
 terraform init
```

Once the initialization completes, Terraform is ready to deploy infrastructure.

Next, step is to plan out your infrastructure with Terraform plan command

```
terraform plan
```

In this case, terraform will look to your configuration and make a plan to deploy with required plugins needs to be installed.

Finally, it’s now the time to apply all the configuration to deploy infrastructure on the target environment, and this can be done as simple as with command

```
terraform apply
```

Now, it’s time to set up infrastructure as Iac with AWS provider.

**Scenario:**
=============

You as a DevOps engineer, your team has decided to launch a web-server in a public subnet with “NginX” using Terraform (Iac) with user data on boot up.

**Prerequisites:**

*   Installed Terraform
*   AWS Account
*   Create access & Secret Access key in your AWS account.
*   [https://registry.terraform.io/providers/hashicorp/aws/latest/docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)

**Let’s get started….**

Alright then, let’s get out basic configurations deployed. Hold on before we do that let setup an Environment.

Create a project directory name **“web-server”** and cd to the directory

**PROVIDER**
============

> **Create main.tf file for configuration**

As, terraform provides various provider access, however we will be using the Amazon Web Services (AWS) for this project. The main.tf file will contain the main set of configuration for your module. 

NB: Hence, it’s not the good practice to setup credential in hard format, there’s another way to do it. We’ll talk about it later in different module. Replace (“ACCESS\_KEY” & “SERET\_KEY”) with your account credentials. If you haven’t created yet please review here [https://docs.aws.amazon.com/powershell/latest/userguide/pstools-appendix-sign-up.html](https://docs.aws.amazon.com/powershell/latest/userguide/pstools-appendix-sign-up.html)

**NETWORKING**
==============

> **Create VPC , Internet\_gateway and subnet.**

In the same main.tf file we will continue with setting up the resources now, as of now we just have finished with providers. We will setup network which may include

*   **_VPC_** with CIDR 10.0.0.0/16 following name label= **“aws\_vpc”**
*   Internet gateway with name label **“aws\_internet\_gateway”** & attribute as **“igw”** and we want to associate our internet gateway with **“aws\_vpc”** what we have just created . Though, inside configuration block of **“igw”** we we’ll provide a single argument **vpc\_id= aws\_vpc.vpc.id** 
as refrence syntax.
*   Now, you might me wondering what arguments and attributes are available for a source. Well to get into you can refer to this [https://registry.terraform.io/providers/hashicorp/aws/latest/docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs) document for AWS.
*   **_1 Public Subnet_** with **CIDR** 10.0.0.0/24 (us-east-1a) , and we will attach subnet with our VPC, and we’re setting up **map\_public\_ip\_on\_launch=true** , so when we spin up an EC2 instance in this subnet it’ll get a Public IP address.

Now, let set up the configuration for Networking resources in the same main.tf

> **Create Route table & associate it with subnet**

Now, getting little more further let’s create a Route Table

*   We’ll create **“aws\_route\_table”** called as **“rtb”**
*   Route Table with name label **“aws\_route\_table”** and we’ll to associate it to our **“aws\_vpc”.**
*   Then we’ll create a nested block inside route table resource. Here we’ll be providing a default route 0.0.0.0/0 and putting it in our internet gateway through which our traffic can go out of our VPC.
*   At last we’ll create **“aws\_route\_table\_association”** called **“rta-subnet1”** i.e= **“aws\_route\_table\_association” “rta-subnet1”.** within this configuration we’ll provide **subnet\_id** and **route\_table\_id** so now there is association between those two resources.

This is will be our last and final setup with Networking. EXCITEDD??

Let’s just do that…….quickly

> **Create Security Group || INBOUND & OUTBOUND Rules**

*   Moving further we’ll create an “aws\_security\_group”.
*   That allows port 80 from anywhere to talk our EC2 instance.
*   We’ll be associate security group to our VPC “**aws\_vpc.vpc.id”**
*   Then , for inbound rule, we’ll setup a single **“ingress”** group to allow incoming traffic from anywhere.
*   Which will be defined as **from\_port =80**, **to\_port =80**, **protocol =”tcp”** and the **cidr\_blocks= \[“0.0.0.0/0”\]** which allows incoming traffic to our intances from anywhere
*   Lastly we’ll define “**egress”** group which AWS by default allows all outgoing traffic from an EC2 Instance.

> **Create an EC2 Instance**

Last, but not the least we’ll be creating an EC2 instance.

*   We’ll create a resource for an EC2 instance with name **“aws\_instance”.**
*   Then we’ll have AMI which uses non-sensitive function to get the AMI Images . We’ll set instance type to **“t2.micro”** in this case we’ll be using free tier available.
*   Finally, we’ll attach our Subnet and Security group to the EC2 instance. Although, we can have 
multiple security groups in the list form separated by comma **\[ “sg1”, “sg2”, “sg3” \]** attached to an EC2. But in this case as we have only one security group created for Port 80. We’ll go single listed.

Whoooooo…..So, we’re done setting up the resources . Now, it’s time to send some user data on EC2 bootup. This is simple bash script with EOF tag, don’t worry if you’re not familiar with EOF it’s just a way for specifying a block of commands.

*   It will install Nginx on EC2 instance,
*   Start Nginx
*   Remove base index.html
*   Will replace it with user data.

> **Deploy your first infrastructure**

**_IMP:: Before going with deployment, make sure you have added you Access & Secret\_Access key to 
the configuration file._**

> **NB: Do not hardcore your access keys in plain formats (other ways you can find documentation, however we’ll explore this in our next module) , as I did is only for demonstration purpose and same will be invalidated by the time you read this article.**

Let’s stick to the topic and deploy our first configuration.

*   Go ahead save the file once you saved your access keys.
*   Now open your Terminal, cd into the project directory.

```
cd web-server
```

Now, initialize terraform with below command

```
sudo terraform init
```

> It will initialize the backend…will download if any required plugins.
>
> Finally, on successful initialization you must see the message “Terraform has been successfully initialized”.
>
> It will create two files in the project directory “web-server” **.terraform.lock.hcl** and **.terraform** which is executable file and will be used to talk to AWS.
>
> Now, as it has been successfully initialized let’s go ahead and Plan terraform.

```
sudo terraform plan -out d3.tfplan
```

As we can see with the output, it’s going to determine what needs to be created mapping with the configuration file. Remember **“+”** sign indicates what resources to be created.

So, if you see at last of the output . We have 7 Resources to get crated to deploy our first web-server with Terraform. Let’s go and apply our first plan saved in the file **d3.tfplan.**

```
sudo terraform apply "d3.tfplan"
```

_It will start creating required resources, this may take while._

Ok, are we done? It was so detailed to get absolute hands-on BTW. Yes we’re done as we can see **“Apply complete!”** 7 Resources has been added . Let’s quickly login to our AWS and check our first web-server deployed in EC2.

Here we’re our web-server has been successfully launched and we can see it has public IPV4 DNS enabled . So let’s go ahead copy and paste in the browser for user data which we had provided in the html. It should pop-up with our customized message.


> **Terraform Destroy**

Best and most fascinating about terraform is we can take down all the infrastructure with just a single command line.

```
terraform destroy
```

Congratulations y’all! You’ve just created an AWS architecture (web-server) as Infrastructure as a 
code using Terraform.

Hope you enjoyed the learning journey. See you soon, in the next module.
