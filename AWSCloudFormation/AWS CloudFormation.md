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
![alt text](/AWSCloudFormation/AWS Lab/StackEvents.JPG)

When the stack status is *CREATE_COMPLETE* , it means the resources have been created.
![alt text](/AWSCloudFormation/AWS%20Lab/CreateStack.JPG)
