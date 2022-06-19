# TOPIC: Deploy-a-high-availability-web-app-using-CloudFormation

> In this project, I'll deploy web servers for a highly available web app using CloudFormation.

![image](https://user-images.githubusercontent.com/40290711/174498986-4b79ef5a-3b31-4d53-8b65-45840dc4b7e4.png)


### Overview:
> AWS CloudFormation is a service that helps you model and set up your AWS resources so that you can spend less time managing those resources and more time focusing on your applications that run in AWS. You create a template that describes all the AWS resources that you want (like Amazon EC2 instances or Amazon RDS DB instances), and CloudFormation takes care of provisioning and configuring those resources for you. You don't need to individually create and configure AWS resources and figure out what's dependent on what; CloudFormation handles that. The following scenarios demonstrate how CloudFormation can help [see](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html).

### Prerequisites:
> AWS Account
> AWS CLI 
> AWS CLI user profile
> Basic understanding of YAML

- Basically, this project is splited into two part namely Networking infrastructure and Server and associated resources section.
- > The Networking section consists of the following:

     > Virtual Private Cloud and subnets.

     > Internet gateway and NAT gateway.

     > Route table.

     > Export the stack output.

- > Server and associated resources section consists of the following:

     > Security groups.
     
     > AutoScaling group.
     
     > Launch configuration.
      
     > Load balancer

### Project Structure 

![image](https://user-images.githubusercontent.com/40290711/174498906-37c4772b-3171-4118-a36d-59a6f83938f8.png)

### Let get Started!!!!

### Step 1

- aws configure
> We have to create a new profile using aws configure command. This require access key and secret key to make profile and this profile will be used in terraform provider for authentication.

```
$ aws configure --profile your Profilename
```

### Step 2

> create a YAML file for our networking resource.
```
touch network.yml
```

> Declear cloudFormation version and Description in the network.yml file
```
AWSTemplateFormatVersion: 2010-09-09
Description: Oloruntobi / Project Two - This template deploys a VPC
```

### Step 3 

> Declear the following parameters.
```
Parameters: 

  EnvironmentName:
    Description: testing 
    Type: String 

  VpcCIDRProject2:
    Default: 10.0.0.0/16 
    Description: testing 
    Type: String  

  PublicSubnet1CIDRProject2:
    Default: 10.0.0.0/24
    Description: testing 
    Type: String 

  PublicSubnet2CIDRProject2:
    Default: 10.0.1.0/24
    Description: testing 
    Type: String     

  PrivateSubnet1CIDRProject2:
    Default: 10.0.2.0/24
    Description: testing 
    Type: String 

  PrivateSubnet2CIDRProject2:
    Default: 10.0.3.0/24
    Description: testing 
    Type: String 
```    
    




