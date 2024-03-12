(Images/LB-HA.png)

# Creating a Highly AVailable Environment

## Introduction

* I chose to study making applications highly available today to better understand how to design resilient architectures on AWS. It helps me improve my skills in creating fault-tolerant systems, aligning with my goal of mastering cloud computing principles for building robust and scalable solutions.

## Prerequisite

* AWS Free tier Account
* Basic knowledge of AWS Management Console, IAM and EC2 services
* Basic knowledge of Networking - security groups, NAT gateway, Internet Gateway
* Knowledge of Load Balancing
* Familiarity with Bash Scripting

## Use Case

* Critical business systems should be deployed as highly available applications—that is, applications remain operational even when some components fail. To achieve high availability in Amazon Web Services (AWS), it is recommended that you run services across *multiple Availability Zones*.

* Many AWS services are inherently highly available, such as load balancers. Many AWS services can also be configured for high availability, such as deploying Amazon Elastic Compute Cloud (Amazon EC2) instances in multiple Availability Zones.


## Cloud Research

* For this challenge, I followed a guided walk-through from AWS re/Start Postgraduate resources on Designing Higly Available Architectures.
* More resources used: 
[Elastic Load Balancing](https://aws.amazon.com/elasticloadbalancing/getting-started/?nc=sn&loc=4)
[What is an Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)
[Getting started with ALB](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/application-load-balancer-getting-started.html)

## Try yourself

✍️ Mini Tutorial

### Step 1 — Configuring your VPC
#### VPC
![VPC Architecture](./Images/VPC.png)
#### Public and private subnets in two Availability Zones
![Subnets](./Images/Task%201-Subnets.png)
#### Route Tables
![Route Tables](./Images/Task%201-Route-Table.png)
#### Network Access Control Lists (NACLs)
![NACLs](./Images/T1-Nacl.png)
![NACLs](./Images/T1-Nacl2.png)
#### An internet gateway that is associated with the public subnets
![Internet Gateway](./Images/T1-IG.png)
#### Security Groups
![All SG](./Images/T1-SG.png)
![Inventory DB SG](./Images/T1-DB%20SG.png)

*The Application Security Group below will be configured later on*

![Inventory App SG](./Images/T1-App%20SG.png)
#### A Network Address Translation (NAT) gateway in one of the public subnets
#### Database Instance
![Inventory DB SG](./Images/T1-DB%20Instance.png)

### Step 2 — Creating an Application Load Balancer

![LB-0](./Images/T2-How%20LB.png)

* On the Services menu, choose **EC2**
* In the left navigation pane, choose **Load Balancers** then **Create load balancer**
* Under **Application Load Balancer**, choose **Create**

![Choose LB](./Images/T2-LB.png)

2.1 - Basic Configuration
![LB-1](./Images/T2-LB-0.png)
2.2 - Network Mapping
![LB-Network](./Images/T2-LB-1.png)
2.3 - Security Groups
![LB-SG](./Images/T2-LB%20SG.png)
2.4 - Listeners and Routing

* In the **Listeners and routing section**, choose **Create target group**

![LB-TG](./Images/T2-TG.png)
![LB-Listeners](./Images/T2-Listeners.png)

2.5 - Load Balancer Created
![LB-Created](./Images/T2-LB-Created%20(2).png)

### Step 3 — Creating an Auto Scaling Group
#### Create an AMI for Auto Scaling
* In the **AWS Management Console**, on the Services  menu, choose **EC2**.
* In the left navigation pane, choose **Instances**.
* In the **Actions**  menu, choose **Image and templates** > **Create image**, then configure:
* Select  Web Server 1.

![ASG-AMI](./Images/T3-AMI-create.png)
![ASG-AMI](./Images/T3-AMI-0.png)

* *Keep all other settings default.*

![ASG-AMI-Created](./Images/T3-AMI-Created.png)
#### Create a Launch Template & Auto Scaling Group
![ASG-LT](./Images/T3-LT-Create.png)
![ASG-LT](./Images/T3-LT-1.png)
![ASG-LT](./Images/T3-LT-2.png)
![ASG-LT](./Images/T3-LT-3.png)
![ASG-LT](./Images/T3-LT-4.png)
* *In the **Advanced Details**, only change the following parts:*

![ASG-LT](./Images/T3-LT-5.png)
![ASG-LT](./Images/T3-LT-5.1.png)
![ASG-LT](./Images/T3-LT-5.2.png)

### Step 4 — Updating Security Groups

![LB-SG](./Images/T4-Update%20SG.png)
#### Load Balancer SG
* We configured the load balancer security group when creating the load balancer. It accepts all incoming *HTTP* and *HTTPS* traffic.
* The load balancer has been configured to forward incoming requests to a *target group*. When Auto Scaling launches new instances, it will automatically add those instances to the target group.

#### Application SG
* Add a rule to accept incoming HTTP traffic from the load balancer.

![LB-SG](./Images/T4-App%20SG.png)
#### Database SG
* Configure the database security group to only accept incoming traffic from the application servers as below

![LB-SG](./Images/T4-DB%20SG.png)

### Step 5 — Testing the Application
* In the left navigation pane, choose **Target Groups**
* Select  **Inventory-App**
* In the lower half of the page, choose the **Targets** tab
* In the **Registered targets** area, occasionally choose the  refresh icon until the **Status** for both instances appears as healthy.
* In the left navigation pane, choose **Load Balancers** and then choose **Inventory-LB**.
* In the **Details** tab in the lower half of the window, copy the **DNS name** to your clipboard.
It should be similar to: *inventory-LB-xxxx.elb.amazonaws.com*

* Open a new web browser tab, paste the DNS name from your clipboard and press ENTER.

![LB-SG](./Images/T5-LB-Test.png)

This is how data flows with a Load Balancer

![LB-SG](./Images/T5-Test-App.png)

### Step 6 — Testing high availabilty
The application is configured to be highly available. You can prove the application's high availability by terminating one of the EC2 instances.

* Return to the EC2 console tab in your web browser (but do not close the web application tab—you will return to it soon).
* In the left navigation pane, choose Instances.
* Select  one of the Inventory-App instances
* Choose Instance State > Terminate instance
* Choose Terminate.
* Return to the web app tab in your web browser and reload  the page several times.
You should notice that the Availability Zone that is shown at the bottom of the page stays the same. Though an instance failed, your application remains available.
*After a few minutes, Amazon EC2 Auto Scaling will also notice the instance failure. It was configured to keep two instances running, so Amazon EC2 Auto Scaling will automatically launch a replacement instance.*

* Return to the EC2 console tab where you have the instances list displayed. In the top-right area, choose the  refresh icon every 30 seconds or so until a new EC2 instance appears.
*After a few minutes, the health check for the new instance should become healthy. The load balancer will resume sending traffic between the two Availability Zones. You can reload your web application tab to see this happen.*

This step demonstrates that your application is now highly available.

## ☁️ Cloud Outcome

* Today's challenge deepened my understanding of creating highly available applications on AWS. Through creating an Application Load Balancer, and setting up an Auto Scaling group, I gained practical knowledge in implementing high availability architectures. 
* Testing for high availability reinforced the importance of redundancy and scalability for uninterrupted service. Overall, this lab provided valuable hands-on experience and insights into building resilient cloud infrastructures

## Next Steps

* Make the Database highly available

## Social Proof

[LinkedIn Post](https://www.linkedin.com/posts/jecinta-atieno_100daysofcloud-awscloud-highavailability-activity-7173424785907924992-Plq8?utm_source=share&utm_medium=member_desktop)
