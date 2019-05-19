## Amazon Virtual Private Cloud (VPC) with AWS CloudFormation
**AWS CloudFormation** gives developers and system administrators an easy way to create and manage a collection of related AWS resources, provisioning and updating them in an orderly and predictable fashion.
You can deploy and update a template and its associated collection of resources (called a **stack**) by using the AWS Management Console, AWS command line interface or APIs. **CloudFormation** is available at no additional charge, and you pay only for the AWS resources needed to run your application.

**Amazon Virtual Private Cloud (Amazon VPC)** lets you provision a logically isolated section of the AWS cloud where you can launch resources within a virtual network. You have complete control over your virtual networking environment, including selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways.
You can customize the network configuration for your virtual private cloud.

For example, you can create a **public-facing subnet** for your web servers that has access to the Internet and place your backend systems such as databases and application servers in a **private facing subnet** with no internet access. You can leverage multiple layers of security, including security groups and network access control lists, to help control access to Amazon EC2 instances in each subnet.  
### Deploy Stack using CloudFormation
1.	Login to **AWS Management Console**
2.	Click **CloudFormation**
3.	Click **Create Stack**,  and upload template vpc-1.yaml
4. Configure
  Stack name: Lab
5. Click **Next** for rest of the steps. We will use the default values.
6. Click **Create Stack**

The stack status will be *CREATE_IN_PROGRESS* until the resources have been deployed.

The **Events** tab previews the work being performed by CloudFormation.  
![alt text](/AWSCloudFormation/AWS%20Lab/StackEvents.JPG)

When the stack status is *CREATE_COMPLETE* , it means the resources have been created.
![alt text](/AWSCloudFormation/AWS%20Lab/CreateStack.JPG)
7. Click on the **Resources** tab. A list of resources is displayed.  
![alt text](/AWSCloudFormation/AWS%20Lab/StackResources.JPG)

### Examine the VPC
The following resources are created by CloudFormation.
1. Amazon VPC
2. Internet Gateway
3. Two subnets
4. Two route tables

All these resources reside within one Availability Zone within a Region.

1. On the **Services** menu, click **VPC**
2. Under **Your VPCs**, *Lab VPC* is created
![alt text](/AWSCloudFormation/AWS%20Lab/LabVPC.JPG)
A VPC is an isolated section of the AWS Cloud that allows resources to communicate with each other and, selectively, with the internet. The **Summary** tab displays the **IPv4 CIDR**, which is the range of IP addresses assigned to the VPC. The VPC has a CIDR of 10.0.0.0/16, which means it contains all addresses that start with *10.0.x.x*

```
AWSTemplateFormatVersion: 2010-09-09
Description: Deploy a VPC

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      Tags:
      - Key: Name
        Value: Lab VPC
```

  * **Type**: Type of resources being created by CloudFormation
  * **CidrBlock**: the IP address range associated with the VPC
  * **EnableDnsHostnames**: Configures the VPC to associate DNS names with Amazon EC2 instances
  * **Tags**: Adds a friendly name to the resource.  

3. In the left navigation pane, click **Internet Gateways**  
![alt text](/AWSCloudFormation/AWS%20Lab/Internetgateway.JPG)  
*An Internet gateway is a horizontally scaled, redundant and highly available component that allows communication between instances in your VPC and the Internet. It therefore imposes no availability risks or bandwidth constraints on the network traffic. It serves two purposes: to provide a target in your VPC route tables for Internet-routable traffic and to perform network address translation (NAT) for instances that have been assigned public IPv4 addresses.  *

```
InternetGateway:
  Type: AWS::EC2::InternetGateway
  Properties:
    Tags:
    - Key: Name
      Value: Lab Internet Gateway

AttachGateway:
  Type: AWS::EC2::VPCGatewayAttachment
  Properties:
    VpcId: !Ref VPC
    InternetGatewayId: !Ref InternetGateway
```
A VPC Gateway Attachment creates a relationship between a VPC and a gateway, such as this Internet Gateway. The !ref makes it easy to link resource reference.

4. In the left navigation pane, click Subnets. There are two subnets:  
![alt text](/AWSCloudFormation/AWS%20Lab/Subnets.JPG)   
  a. **Public Subnet 1** is connected to the Internet via the Internet Gateway and can be used by resources that need to be publicly accessible.  
  b. **Private Subnet 1** is not connected to the Internet. Resources in this subnet cannot be reached from the Internet.

```
PublicSubnet1:
  Type: AWS::EC2::Subnet
  Properties:
    VpcId: !Ref VPC
    CidrBlock: 10.0.0.0/24
    AvailabilityZone: !Select
      - '0'
      - !GetAZs ''
    Tags:
      - Key: Name
        Value: Public Subnet 1

PrivateSubnet1:
  Type: AWS::EC2::Subnet
  Properties:
    VpcId: !Ref VPC
    CidrBlock: 10.0.1.0/24
    AvailabilityZone: !Select
      - '0'
      - !GetAZs ''
    Tags:
      - Key: Name
        Value: Private Subnet 1
```

The properties are:
  * **VpcId** refers to the VPC that contains the subnets
  *  **CidrBlock** is the range of IP address assigned to the subnets
  * **Availability Zone** defines which physical location within the Region should contain the subent.

*Note that the Availability Zone is using a function called !Select and a function called !GetAZs. The code is retrieving a list of Availability Zones within the region and is referencing the first element from the list. In this manner, the template can be used in any region because it retrieves the list of Availability Zones at runtime rather than having the Availability Zones hard coded in the template*

5. In the left navigation pane, click **Route Tables**  
  * Select the **Public Route Table**

![alt text](/AWSCloudFormation/AWS%20Lab/RouteTables.JPG)

Route tables are used to route traffic in and out of traffic locally. The configuration for this route table is:
  * For traffic within the VPC (10.0.0.0/16), route the traffic logically
  * For traffic going to the Internet (0.0.0.0/0), route the traffic to the Internet Gateway.

  ```
  PublicRouteTable:
  Type: AWS::EC2::RouteTable
  Properties:
    VpcId: !Ref VPC
    Tags:
      - Key: Name
        Value: Public Route Table
  ```

  Here is the code that defined the route to the Internet within the Public Route Table.

  ```
  PublicRoute:
  Type: AWS::EC2::Route
  Properties:
    RouteTableId: !Ref PublicRouteTable
    DestinationCidrBlock: 0.0.0.0/0
    GatewayId: !Ref InternetGateway
```

The configuration for the route is:
  * **RouteTableId** indicates the route table that owns the route.
  * **DestinationCidrBlock** defines the IP address range for this routing rule.
  * **GatewayId** defines where to route the traffic, which in this case is the Internet Gateway.

6. Click on the **Subnet Associations** tab.

![alt text](/AWSCloudFormation/AWS%20Lab/SubnetAssociation.JPG)
```
PublicSubnetRouteTableAssociation1:
  Type: AWS::EC2::SubnetRouteTableAssociation
  Properties:
    SubnetId: !Ref PublicSubnet1
    RouteTableId: !Ref PublicRouteTable
```
The console shows that the Public Route Table is associated with **Public Subnet 1.** A route table can be associated with multiple subnets, with each association requiring an explicit linkage.

7. **Stack output** : The CloudFormation template has been configured to return information about the resources it created:
  * **VPC** is the ID of the VPC that was created
  * **AZ1** shows the Availability Zone in which subnets were created

![alt text](/AWSCloudFormation/AWS%20Lab/StackOutputs.JPG)

```
Outputs:
  VPC:
    Description: VPC
    Value: !Ref VPC

  AZ1:
    Description: Availability Zone 1
    Value: !GetAtt
      - PublicSubnet1
      - AvailabilityZone
```
The AZ1 output uses the **!GetAtt** function to retrieve an attribute of the resources. In this case, it is retrieving the Availability Zone attribute from the Public Subnet 1.

### Updating a Stack
Once a CloudFormation stack has been deployed, it is recommended that any changes to the resources should be made through CloudFormation rather than by directly modifying the resources.

1. In the **Action** menu, click **Update Stack**, then configure
      * Replace current template> Upload a template file> Choose file > [vpc-2.yaml](/AWSCloudFormation/vpc-2.yaml)
      * Click Next> Next to accept default Options
      * Click Update Stack  
![alt text](/AWSCloudFormation/AWS%20Lab/UpdateStack.JPG)  
2. Click Events tab > Stack Info Tab to check status      
![alt text](/AWSCloudFormation/AWS%20Lab/Update%20Complete.JPG)
3. Click Services> VPC> subnets. Four subnets are now displayed.   
![alt text](/AWSCloudFormation/AWS%20Lab/UpdateSubnets.JPG)  
![alt text](/AWSCloudFormation/AWS%20Lab/Updatedsubnetassociatons.JPG)  
The VPC has now been updated to support Highly Available applications.

### View Stack in CloudFormation Designer
 AWS CloudFormation Designer (Designer) is a graphic tool for creating, viewing, and modifying AWS CloudFormation templates. With Designer, you can diagram your template resources using a drag-and-drop interface, and then edit their details using the integrated JSON and YAML editor. Whether you are a new or an experienced AWS CloudFormation user, AWS CloudFormation Designer can help you quickly see the interrelationship between a template's resources and easily modify templates.

 1. Click Services> CloudFormation> Lab Stack> Template tab> View in Designer
![alt text](/AWSCloudFormation/AWS%20Lab/CloudFormationtemplate.JPG)

The lower portion of the window displays the code within the template that defines the resource.  

### Delete Stack
1. Click Lab stack> Actions> Delete Stack
2. Click the Events tab to view details of Deletion  
![alt text](/AWSCloudFormation/AWS%20Lab/DeleteStack.JPG)
3. On the Services menu, click VPC. The **Lab VPC** is no longer listed  
![alt text](/AWSCloudFormation/AWS%20Lab/DeletedVPC.JPG)



Credits: [AWS Training and Certification](https://www.qwiklabs.com/focuses/363?parent=catalog)
© 2019 Amazon Web Services, Inc. and its affiliates. All rights reserved. This work may not be reproduced or redistributed, in whole or in part, without prior written permission from Amazon Web Services, Inc. Commercial copying, lending, or selling is prohibited.
