# TOPIC: Deploy-a-high-availability-web-app-using-CloudFormation

> In this project, I'll deploy web servers for a highly available web app using CloudFormation.

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

- > Server and associated resources section consists of the following:

     > Security groups.
     
     > AutoScaling group.
     
     > Launch configuration.
      
     > Load balancer



