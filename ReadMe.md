### Working with the ELB.

Lab Overview
This lab introduces the concept of Elastic Load Balancing. In this lab you will use Elastic Load Balancing to load balance traffic across multiple Amazon Elastic Compute Cloud (EC2) instances in a single Availability Zone. You will deploy a simple application on multiple Amazon EC2 instances and observe load balancing by viewing the application in your browser.

First, you will launch a pair of instances, bootstrap them to install web servers and content, and then access the instances independently using Amazon EC2 DNS records. Next, you will set up Elastic Load Balancing, add your instances to the load balancer, and then access the DNS record again to watch your requests load balance between servers. Finally, you will view Elastic Load Balancing metrics in Amazon CloudWatch.

The following diagram provides a high-level overview of the architecture you will implement in this exercise.

diagram

Topics covered
This lab will take you through:

Launching a multiple server web farm on Amazon EC2.
Using bootstrapping techniques to configure Linux instances with Apache, PHP, and a simple PHP application downloaded from Amazon Simple Storage Service (S3).
Creating and configuring a load balancer that will sit in front of your Amazon EC2 web server instances.
Viewing Amazon CloudWatch metrics for the load balancer.
Technical knowledge prerequisites
To successfully complete this lab, you should be familiar with the AWS Management Console.

Elastic Load Balancing

Elastic Load Balancing automatically distributes incoming application traffic across multiple Amazon EC2 instances. It enables you to achieve greater levels of fault tolerance in your applications, seamlessly providing the required amount of load balancing capacity needed to distribute application traffic.

Achieve higher levels of fault tolerance for your applications by using Elastic Load Balancing to automatically route traffic across multiple instances and multiple Availability Zones. Elastic Load Balancing ensures that only healthy Amazon EC2 instances receive traffic by detecting unhealthy instances and rerouting traffic across the remaining healthy instances. If all of your Amazon EC2 instances in one Availability Zone are unhealthy, and you have set up Amazon EC2 instances in multiple Availability Zones, Elastic Load Balancing will route traffic to your healthy Amazon EC2 instances in those other zones.

Elastic Load Balancing automatically scales its request-handling capacity to meet the demands of application traffic. Additionally, Elastic Load Balancing offers integration with Auto Scaling to ensure that you have back-end capacity to meet varying levels of traffic levels without requiring manual intervention.

Elastic Load Balancing works with Amazon Virtual Private Cloud (VPC) to provide robust networking and security features. You can create an internal (non-Internet facing) load balancer to route traffic using private IP addresses within your virtual network. You can implement a multi-tiered architecture using internal and Internet-facing load balancers to route traffic between application tiers. With this multi-tier architecture, your application infrastructure can use private IP addresses and security groups, allowing you to expose only the Internet-facing tier with public IP addresses.

Elastic Load Balancing provides integrated certificate management and SSL decryption, allowing you to centrally manage the SSL settings of the load balancer and offload CPU-intensive work from your instances.

This lab guide explains basic concepts of Elastic Load Balancing in a step-by-step fashion. However, it can only give a brief overview of Elastic Load Balancing concepts. For further information, see http://aws.amazon.com/elasticloadbalancing/.

Start Lab
At the top of your screen, launch your lab by choosing Start Lab
This starts the process of provisioning your lab resources. An estimated amount of time to provision your lab resources is displayed. You must wait for your resources to be provisioned before continuing.

 If you are prompted for a token, use the one distributed to you (or credits you have purchased).

Open your lab by choosing Open Console
This automatically logs you in to the AWS Management Console.

 Do not change the Region unless instructed.

Common Login Errors
Error: Federated login credentials



If you see this message:

Close the browser tab to return to your initial lab window
Wait a few seconds
Choose Open Console again
You should now be able to access the AWS Management Console.

Error: You must first log out



If you see the message, You must first log out before logging into a different AWS account:

Choose here
Close your browser tab to return to your initial lab window
Choose Open Console again
Task 1: Launch Web Servers
In this task, you will launch two Amazon Linux EC2 instances, with an Apache PHP web server and basic application installed on initialization. You will also demonstrate a simple example of bootstrapping instances using the Amazon EC2 metadata service.

Amazon Machine Images (AMIs) and instances

Amazon EC2 provides templates known as Amazon Machine Images (AMIs) that contain a software configuration (for example, an operating system, an application server, and applications). You use these templates to launch an instance, which is a copy of the AMI running as a virtual server in the cloud.

You can launch different types of instances from a single AMI. An instance type essentially determines the hardware capabilities of the virtual host computer for your instance. Each instance type offers different compute and memory capabilities. Select an instance type based on the amount of memory and computing power that you need for the application or software that you plan to run on the instance. You can launch multiple instances from an AMI.

Your instance keeps running until you stop or terminate it, or until it fails. If an instance fails, you can launch a new one from the AMI.

When you create an instance, you will be asked to select an instance type. The instance type you choose determines how much throughput and processing cycles are available to your instance.

On the Services menu, click EC2.

In the left navigation pane, click Instances.

Click Launch Instances

In the Name and tags section:

Name : 
MyLBInstances
This name, more correctly known as a tag, will appear in the console when the instance launches. It makes it easy to keep track of running machines in a complex environment. Use a name that you can easily recognize and remember.

In Application and OS Images (Amazon Machine Image) , Quick Start sub-section click Amazon Linux.

Latest Amazon Linux 2 AMI shall be selected automatically.

The t2.micro instance type, which is the lowest-cost option, should be automatically selected.

In the Key pair (login) section:
Select Proceed without a key pair (Not recommended).
In the Network settings section, click Remove Edit:
VPC - required : Lab VPC
Firewall (security groups) : Create security group
Security group name - required : 
MyLBInstances
Description - required : 
MyLBInstances
A security group acts as a firewall that controls the traffic allowed into a group of instances. When you launch an Amazon EC2 instance, you can assign it to one or more security groups. For each security group, you add rules that govern the allowed inbound traffic to instances in the group. All other inbound traffic is discarded. You can modify rules for a security group at any time. The new rules are automatically enforced for all existing and future instances in the group.

By default, AWS creates a rule that allows Secure Shell (SSH) access from any IP address. It is highly recommended that you restrict terminal access to the ranges of IP addresses (e.g., IPs assigned to machines within your company) that have a legitimate business need to administer your Amazon EC2 instance.

Click Remove to remove the SSH rule.

Click Add security group rule then configure:

Type: HTTP
Source type: Anywhere
This will add a default handler for HTTP that will allow requests from anywhere on the Internet. Since you want this web server to be accessible to the general public.

In the Advanced Details section

Enter the following into User data:

#!/bin/sh
yum -y install httpd php
chkconfig httpd on
systemctl start httpd.service
cd /var/www/html
wget https://us-west-2-aws-training.s3.amazonaws.com/courses/spl-03/v4.3.14.prod-6ca03e32/scripts/examplefiles-elb.zip
unzip examplefiles-elb.zip

content_copy
This will allow you to bootstrap your instance. It will install Apache and PHP and sample code (PHP scripts) needed for this lab when the instance is created and launched. User data provides a mechanism to pass data or a script to the Amazon metadata service, which instances can access at launch time.

Tip If you type this text instead of copying it, press SHIFT+ENTER to create new lines in the text box.

In the Configure storage section, you will use the default storage device configuration.

In the Summary section:

Number of instances: 
2
Review your choices, and then click Launch instance
 You may see a warning on this screen that “Your security group … is open to the world.” This is a result of not restricting SSH access to your machine, as described earlier. For the purposes of this lab only, you may ignore this warning.

 Please do not launch your instance with a key pair.

A status page notifies you that your instances are launching.

Click View all instances

Wait for your instances to display:

Instance State:  running
Status Checks:  2/2 checks passed
This indicates that your instance is now fully available.

 This may take a few minutes. You can refresh the status of your instances by clicking the refresh  icon.

Task 2: Connect to Each Web Server
In this task, you will:

Retrieve the public DNS entries for both of your EC2 instances
Point your browser at each DNS entry to access your instances web server
All Amazon EC2 instances are assigned two IP addresses at launch: a private IP address (RFC 1918) and a public IP address that are directly mapped to each other through Network Address Translation (NAT). Private IP addresses are only reachable from within the Amazon EC2 network. Public addresses are reachable from the Internet.

Amazon EC2 also provides an internal DNS name and a public DNS name that map to the private and public IP addresses, respectively. The internal DNS name can only be resolved within Amazon EC2. The public DNS name resolves to the public IP address outside the Amazon EC2 network and to the private IP address within the Amazon EC2 network.

Select  your first Amazon EC2 instance.

Copy the Public DNS (IPv4) value to your Clipboard.

It will look something like ec2-54-84-236-205.compute-1.amazonaws.com.

Open a new browser window, paste the Public DNS value into the address bar, and press ENTER. Your browser will display a screen like this:
logo

This is the web page returned by the PHP script that was installed when the instance was started. It is a simple script that interrogates the metadata service on each machine and returns the instance ID and the name of the Availability Zone in which the instance is running.

Repeat the previous two steps for your second instance.
Notice that each machine displays a different instance ID. This will help you identify which instance is processing your request when you put a load balancer in front of them.

If you see an error instead of the instance ID and Availability Zone when you access the instances from the browser, try again after a couple of minutes. It’s possible that the bootstrapping script is still running and has not yet completed installing and starting the web server and PHP application. If errors persist, verify that you entered the bootstrap script correctly when you launched your instances and that the security group has port 80 open.

Task 3: Create a Load Balancer
You now have two web servers. Now you need a load balancer in front of these servers to give your users a single location for accessing both and to balance user requests across them. In this task, you will be create a simple HTTP application load balancer.

In the AWS Management Console, on the left navigation pane, click Load Balancers.

Click Create Load Balancer

Below Application Load Balancer, click Create then configure:

Below Basic configuration :

Load balancer name: 
ELB
Below Network mapping :
VPC: Lab VPC
Select  all of the availability zones under Mappings
Below Security groups:
Delete  default
Select  MyLBInstances
Below Listeners and routing: click on 
Create target group
In the next tab, configure following details
Target group name: 
myTarget
Expand  Advanced health check settings then configure:
Healthy threshold: 
3
Click Next
The ELB will periodically test the ping path on each of your web service instances to determine health: a 200 HTTP response code indicates a healthy status, and any other response code indicates an unhealthy status. If an instance is unhealthy and continues in that state for a successive number of checks (unhealthy threshold), the load balancer will remove it from service until it recovers.

In this configuration, making the ping path just a slash (/) will return the default page—the PHP-generated page seen earlier.

The healthy threshold is the number of successful checks the load balancer expects to see in a row before bringing an instance into service behind the load balancer. The lower value will speed things up for this exercise.

Select  both of your instances.

Click on 
Include as pending below

Click Create target group

Come back to the orginal load balancer tab

Click on the refresh button within Listeners and routing

Select myTarget from dropdown menu

Click Create load balancer

Click View load balancer

It will take a couple of minutes to start up the load balancer, attach your web servers, and pass the health checks.

Wait for the State to display Active.

Copy the DNS name to your clipboard.

Open a new browser tab and then:

Paste the DNS name
Press Enter
Refresh your browser a few times.
You will notice that the Amazon EC2 Instance IDs change. This means that the repeated responses are coming back through your two different web servers.

Load balancers can span Availability Zones, and they also scale elastically as needed to handle demand. Therefore, you should always access a load balancer by DNS hostname, and not by IP address. A load balancer may have multiple IP addresses associated with its DNS hostname.

In the AWS Management Console, in the left navigation pane, click Target Groups
 myTarget should be selected.

Click the Targets tab.
Notice that both of your instances are healthy.

Task 4: View Elastic Load Balancing CloudWatch Metrics
In this task, you will use Amazon CloudWatch to monitor your ELB. Elastic Load Balancing automatically reports load balancer metrics to CloudWatch.

Amazon CloudWatch provides monitoring for AWS cloud resources and the applications customers run on AWS. Developers and system administrators can use it to collect and track metrics, gain insight, and react immediately to keep their applications and businesses running smoothly. CloudWatch monitors AWS resources such as Amazon EC2 and Amazon Relational Database Service (RDS) DB instances and can also monitor custom metrics generated by your applications and services. With CloudWatch, you gain system-wide visibility into resource use, application performance, and operational health.

Amazon CloudWatch provides a reliable, scalable, and flexible monitoring solution that you can start using within minutes. You no longer need to set up, manage, or scale your own monitoring systems and infrastructure. Using CloudWatch, you can easily monitor as much or as little metric data as you need. CloudWatch lets you programmatically retrieve your monitoring data, view graphs, and set alarms to help you troubleshoot, spot trends, and take automated action based on the state of your cloud environment.

In the AWS Management Console, on the Services menu, click CloudWatch.

In the left navigation pane, click Metrics.

Click on All metrics

Click ApplicationELB

Explore the metrics that are being reported to CloudWatch.

It may take upto 5 minutes for the metrics to be populated

Load balancing metrics include latency, request count, and healthy and unhealthy host counts. Metrics are reported as they are encountered and can take several minutes to show up in CloudWatch.

End Lab
Follow these steps to close the console, end your lab, and evaluate the experience.

Return to the AWS Management Console.

On the navigation bar, choose awsstudent@<AccountNumber>, and then choose Sign Out.

Choose End Lab

Choose OK

(Optional):

Select the applicable number of stars 
Type a comment
Choose Submit

1 star = Very dissatisfied
2 stars = Dissatisfied
3 stars = Neutral
4 stars = Satisfied
5 stars = Very satisfied
You may close the window if you don't want to provide feedback.

Conclusion
 Congratulations! You now have successfully:

Launched a multiple server web farm on Amazon EC2
Used bootstrapping techniques to configure Linux instances with Apache, PHP, and a simple PHP application
Created and configured a load balancer that sits in front of your Amazon EC2 web server instances
Viewed Amazon CloudWatch metrics for the load balancer
