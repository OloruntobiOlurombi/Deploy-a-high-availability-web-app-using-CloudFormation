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

> create a YAML file for our networking resource and a JSON file for the parameters.
```
touch network.yml parameters.json 
```
> Declear our parameters in the JSON file

```
[
    {
        "ParameterKey": "EnvironmentName",
        "ParameterValue": "Project2"
    },
    {
        "ParameterKey": "VpcCIDRProject2",
        "ParameterValue": "10.0.0.0/16"
    },
    {
        "ParameterKey": "PublicSubnet1CIDRProject2",
        "ParameterValue": "10.0.0.0/24"
    },
    {
        "ParameterKey":"PublicSubnet2CIDRProject2",
        "ParameterValue":"10.0.1.0/24"
    },
    {
        "ParameterKey": "PrivateSubnet1CIDRProject2",
        "ParameterValue": "10.0.2.0/24"
    },
    {
        "ParameterKey": "PrivateSubnet2CIDRProject2",
        "ParameterValue": "10.0.3.0/24"
    }
]
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
### Step 4 

> Create Networking infrastructure Resources in the network.yml
``` 

Resources:
# Create VPC 
  UdacityVPC2:
    Type: 'AWS::EC2::VPC'
    Properties:
      CidrBlock: !Ref VpcCIDRProject2
      EnableDnsHostnames: 'true'
      Tags:
      - Key: name
        Value: !Ref EnvironmentName

# Create an Internet Gateway for our public subnets 
  myInternetGateway:
    Type: 'AWS::EC2::InternetGateway'
    Properties:
      Tags:
      - Key: Name 
        Value: !Ref EnvironmentName   

# Create an Attachment for our Internet Gateway 
  myInternetGatewayAttachment:
    Type: 'AWS::EC2::VPCGatewayAttachment'
    Properties:
      VpcId: !Ref UdacityVPC2
      InternetGatewayId: !Ref myInternetGateway    

#  Create the First Public Subnet for our server
  PublicSubnet1: 
    Type: 'AWS::EC2::Subnet'
    Properties: 
      VpcId: !Ref UdacityVPC2
      AvailabilityZone: !Select [0, !GetAZs '']
      CidrBlock: !Ref PublicSubnet1CIDRProject2
      MapPublicIpOnLaunch: true 
      Tags:
        - Key: Name 
          Value: !Sub ${EnvironmentName} Public Subnet (AZ1)   

# Create the Second Public Subnet for our server              
  PublicSubnet2: 
    Type: 'AWS::EC2::Subnet' 
    Properties: 
      VpcId: !Ref UdacityVPC2
      AvailabilityZone: !Select [1, !GetAZs '']
      CidrBlock: !Ref PublicSubnet2CIDRProject2
      MapPublicIpOnLaunch: true 
      Tags:
        - Key: Name 
          Value: !Sub ${EnvironmentName} Public Subnet (AZ2)

# Create the First Private Subnet for our server 
  PrivateSubnet1: 
    Type: 'AWS::EC2::Subnet' 
    Properties:
      VpcId: !Ref UdacityVPC2
      AvailabilityZone: !Select [0, !GetAZs '']
      CidrBlock: !Ref PrivateSubnet1CIDRProject2
      MapPublicIpOnLaunch: false 
      Tags:
        - Key: Name 
          Value: !Sub ${EnvironmentName} Private Subnet (AZ1)    

# Create the First Private Subnet for our server 

  PrivateSubnet2: 
    Type: 'AWS::EC2::Subnet' 
    Properties:
      VpcId: !Ref UdacityVPC2
      AvailabilityZone: !Select [1, !GetAZs '']
      CidrBlock: !Ref PrivateSubnet2CIDRProject2
      MapPublicIpOnLaunch: false 
      Tags: 
        - Key: Name 
          Value: !Sub ${EnvironmentName} Private Subnet (AZ2)

# Create the first Elastic IP Address for the NatGateway   

  NatGateway1EIP:
    Type: 'AWS::EC2::EIP' 
    DependsOn: myInternetGatewayAttachment
    Properties:
      Domain: vpc 

# Create the Second Elastic IP Address for the NatGateway  

  NatGateway2EIP:
    Type: 'AWS::EC2::EIP' 
    DependsOn: myInternetGatewayAttachment
    Properties: 
      Domain: vpc 

# Create the First Natgateway for the public subnet 

  NatGateway1: 
    Type: 'AWS::EC2::NatGateway' 
    Properties: 
      AllocationId: 
        Fn::GetAtt:
        - NatGateway1EIP
        - AllocationId
      SubnetId: 
        Ref: PublicSubnet1 
      Tags:
      - Key: Name 
        Value: !Ref EnvironmentName 

# Create the Second Natgateway for the public subnet         
  NatGateway2: 
    Type: 'AWS::EC2::NatGateway' 
    Properties: 
      AllocationId:
        Fn::GetAtt:
        - NatGateway2EIP
        - AllocationId    
      SubnetId: 
        Ref: PublicSubnet2 
      Tags: 
      - Key: Name 
        Value: !Ref EnvironmentName 

# Create a public route table for our VPC
  PublicRouteTable: 
    Type: 'AWS::EC2::RouteTable'  
    Properties: 
      VpcId: !Ref UdacityVPC2
      Tags: 
        - Key: Name 
          Value: !Sub ${EnvironmentName} Public Routes  

# Create a default Public Route for our Route Table
  DefaultPublicRoute:
    Type: 'AWS::EC2::Route' 
    DependsOn: myInternetGatewayAttachment
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref myInternetGateway   

# Create a subnet route table association for our first public subnet 1 
  PublicSubnet1RouteTableAssociation:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSubnet1 

#  Create a subnet route table association for our second public subnet 1   

  PublicSubnet2RouteTableAssociation:
    Type: 'AWS::EC2::SubnetRouteTableAssociation' 
    Properties: 
      RouteTableId: !Ref PublicRouteTable 
      SubnetId: !Ref PublicSubnet2

# Create the first private route table for our VPC      

  PrivateRouteTable1:
    Type: 'AWS::EC2::RouteTable'
    Properties:
      VpcId: !Ref UdacityVPC2
      Tags: 
        - Key: Name 
          Value: !Sub ${EnvironmentName} Private Routes 1 

# Create the first default Private Route for our Route Table    

  DefaultPrivateRoute1:
    Type: 'AWS::EC2::Route' 
    Properties: 
      RouteTableId: !Ref PrivateRouteTable1
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NatGateway1 

#  Create a subnet route table association for our first private subnet 1   

  PrivateSubnet1RouteTable1Association:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'  
    Properties:
      RouteTableId: !Ref PrivateRouteTable1
      SubnetId: !Ref PrivateSubnet1  

# Create the second private route table for our VPC   

  PrivateRouteTable2:
    Type: 'AWS::EC2::RouteTable'  
    Properties:
      VpcId: !Ref UdacityVPC2
      Tags: 
        - Key: Name 
          Value: !Sub ${EnvironmentName} Private Routes 2   

# Create the second default Private Route for our Route Table         

  DefaultPrivateRoute2:
    Type: 'AWS::EC2::Route'
    Properties:
      RouteTableId: !Ref PrivateRouteTable2 
      DestinationCidrBlock: 0.0.0.0/0 
      NatGatewayId: !Ref NatGateway2       

#  Create a subnet route table association for our second private subnet 1   

  PrivateSubnet2RouteTable2Association:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      RouteTableId: !Ref PrivateRouteTable2
      SubnetId: !Ref PrivateSubnet2   
``` 

### Step 5

> Create Networking infrastructure Outputs in the network.yml

```
Outputs:
    
  UdacityVPC: 
    Description: A reference to the created VPC 
    Export:
      Name: !Sub ${EnvironmentName}-VPCID
    Value: !Ref UdacityVPC2  

  VPCPublicRouteTable:
    Description: Public Routing 
    Value: !Ref PublicRouteTable
    Export:
      Name: !Sub ${EnvironmentName}-PUB-RT

  VPCPrivateRouteTable1:
    Description: Private Routing AZ1 
    Value: !Ref PrivateRouteTable1
    Export: 
      Name: !Sub ${EnvironmentName}-PRI1-RT

  VPCPrivateRouteTable2:
    Description: Private Routing AZ2 
    Value: !Ref PrivateRouteTable2
    Export: 
      Name: !Sub ${EnvironmentName}-PRI2-RT 

  PublicSubnets: 
    Description: A list of the public subnets 
    Value: !Join [ ",", [ !Ref PublicSubnet1, !Ref PublicSubnet2 ]]
    Export:
      Name: !Sub ${EnvironmentName}-PUB-NETS

  PrivateSubnets: 
    Description: A list of the private subnets 
    Value: !Join [ ",", [ !Ref PrivateSubnet1, !Ref PrivateSubnet2 ]]
    Export: 
      Name: !Sub ${EnvironmentName}-PRIV-NETS     

  PublicSubnet1: 
    Description: A reference to the public subnet in the 1st Availability Zone 
    Value: !Ref PublicSubnet1 
    Export: 
      Name: !Sub ${EnvironmentName}-PUB1-SN 

  PublicSubnet2: 
    Description: A reference to the public subnet in the 2nd Availability Zone
    Value: !Ref PublicSubnet2 
    Export: 
      Name: !Sub ${EnvironmentName}-PUB2-SN

  PrivateSubnet1: 
    Description: A reference to the private subnet in the 1st Availability Zone
    Value: !Ref PrivateSubnet1 
    Export: 
      Name: !Sub ${EnvironmentName}-PRI1-SN

  PrivateSubnet2: 
    Description: A reference to the private subnet in the 2nd Availability Zone
    Value: !Ref PrivateSubnet2
    Export: 
      Name: !Sub ${EnvironmentName}-PRI2-SN                 
``` 

### Step 6

> create a YAML file for our server resource and a JSON file for the parameters.
```
touch server.yml server-parameters.json
```
> Declear our parameters in the JSON file

``` 
[
    {
        "ParameterKey": "EnvironmentName",
        "ParameterValue": "Project2"
    }
]
```

> Declear cloudFormation version and Description in the server.yml file
```
AWSTemplateFormatVersion: 2010-09-09
Description: Oloruntobi / Project Two - This template deploys Servers Resources

Parameters:

  EnvironmentName:
    Description: testing 
    Type: String 
```

### Step 7

> Declear the following parameters.
```
Parameters:

  EnvironmentName:
    Description: testing 
    Type: String 
```    

### Step 8

> Create Networking infrastructure Resources in the network.yml
``` 
Resources:

  # IAM Role to allow EC2 Session Manager to access our server
  # AWS::IAM::Role
  RoleForSSMAccess:
    Type: 'AWS::IAM::Role'
    Properties: 
      AssumeRolePolicyDocument: 
        Version: "2012-10-17"
        Statement: 
          - Effect: Allow 
            Principal: 
              Service: 
                - ec2.amazonaws.com 
            Action: 
              - 'sts:AssumeRole'
      Path: / 
      Policies: 
        - PolicyName: root 
          PolicyDocument: 
            Version: "2012-10-17"
            Statement: 
              - Effect: Allow 
                Action: '*'
                Resource: '*'

# Instance Profile 
# AWS::IAM::InstanceProfile 

  ServerInstanceProfile: 
    Type: 'AWS::IAM::InstanceProfile'
    Properties:
      Path: / 
      Roles: 
        - !Ref RoleForSSMAccess 

# Create Security Group for the Load Balancer
# AWS::EC2::SecurityGroup
  LBSecGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow http to our load balancer
      VpcId:
        Fn::ImportValue:
          !Sub "${EnvironmentName}-VPCID"
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 80
        ToPort: 80
        CidrIp: 0.0.0.0/0
      SecurityGroupEgress:
      - IpProtocol: tcp
        FromPort: 80
        ToPort: 80
        CidrIp: 0.0.0.0/0
      #- IpProtocol: tcp 
       # FromPort: 8080 
        #ToPort: 8080 
        #CidrIp: 0.0.0.0/0   

# Create Security Group for the Web Server
# AWS::EC2::SecurityGroup
  WebServerSecGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow http to our hosts and SSH from local only
      VpcId:
        Fn::ImportValue:
          !Sub "${EnvironmentName}-VPCID"
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 80
        ToPort: 80
        CidrIp: 0.0.0.0/0
      - IpProtocol: tcp
        FromPort: 22
        ToPort: 22
        CidrIp: 0.0.0.0/0
      SecurityGroupEgress:
      - IpProtocol: tcp
        FromPort: 0
        ToPort: 65535
        CidrIp: 0.0.0.0/0  

# Create a Launch Configuration for the Web Instance
# AWS::AutoScaling::LaunchConfiguration
  WebAppLaunchConfig:
    Type: AWS::AutoScaling::LaunchConfiguration
    Properties: 
      UserData: 
        Fn::Base64: !Sub | 
          #!/bin/bash
          apt-get update -y
          apt-get install apache2 -y
          systemctl start apache2.service
          cd /var/www/html
          echo "Udacity Demo Web Server Up and Running!" > index.html  
      ImageId: ami-0df32f8302dfe67df
      SecurityGroups: 
      - Ref: WebServerSecGroup 
      InstanceType: t3.small 
      BlockDeviceMappings: 
      - DeviceName: "/dev/sdk"
        Ebs: 
          VolumeSize: '10'

# Create an Autoscaling Group for the Launch Configuration
# AWS::AutoScaling::AutoScalingGroup 
  WebAppGroup: 
    Type: AWS::AutoScaling::AutoScalingGroup 
    Properties: 
      VPCZoneIdentifier: 
      - Fn::ImportValue: 
          !Sub "${EnvironmentName}-PRIV-NETS"  
      LaunchConfigurationName:
        Ref: WebAppLaunchConfig 
      MinSize: '3'
      MaxSize: '5'
      TargetGroupARNs: 
      - Ref: WebAppTargetGroup 
      HealthCheckGracePeriod: 60 
      HealthCheckType: ELB   

# Create an Elastic Load Balancer for our Public Subnets
# AWS::ElasticLoadBalancingV2::LoadBalancer 
  WebAppLB: 
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer 
    Properties: 
      Subnets: 
      - Fn::ImportValue: !Sub "${EnvironmentName}-PUB1-SN"
      - Fn::ImportValue: !Sub "${EnvironmentName}-PUB2-SN" 
      SecurityGroups: 
      - Ref: LBSecGroup 

# Create a Listener for the Target Group and Load Balancer
# AWS::ElasticLoadBalancingV2::Listener 
  Listener:
    Type: AWS::ElasticLoadBalancingV2::Listener 
    Properties: 
      DefaultActions: 
      - Type: forward 
        TargetGroupArn: 
          Ref: WebAppTargetGroup 
      LoadBalancerArn: 
        Ref: WebAppLB
      Port: '80'
      Protocol: HTTP  

# Create a Listener Rule
# AWS::ElasticLoadBalancingV2::ListenerRule 
  ALBListenerRule: 
    Type: AWS::ElasticLoadBalancingV2::ListenerRule 
    Properties: 
      Actions: 
      - Type: forward 
        TargetGroupArn: !Ref 'WebAppTargetGroup'
      Conditions: 
      - Field: path-pattern 
        Values: [/]
      ListenerArn: !Ref 'Listener'   
      Priority: 1

# Create a Target Group 
# AWS::ElasticLoadBalancingV2::TargetGroup 
  WebAppTargetGroup: 
    Type: AWS::ElasticLoadBalancingV2::TargetGroup 
    Properties:
      HealthCheckIntervalSeconds: 10 
      HealthCheckPath: / 
      HealthCheckProtocol: HTTP 
      HealthCheckTimeoutSeconds: 8 
      HealthyThresholdCount: 2 
      Port: 80 
      Protocol: HTTP 
      UnhealthyThresholdCount: 5 
      VpcId: 
        Fn::ImportValue:
          Fn::Sub: "${EnvironmentName}-VPCID"           

``` 
