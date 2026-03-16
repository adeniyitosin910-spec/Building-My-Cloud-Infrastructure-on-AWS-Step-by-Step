# Building-My-Cloud-Infrastructure-on-AWS-Step-by-Step1. Planning the Infrastructure

Before I got started, I took some time to think through the architecture I wanted to build:

**VPC** – A Virtual Private Cloud to keep my network isolated and secure.

**Subnets** – I needed to separate public-facing resources from private ones.

**Internet Gateway** – This would give my public resources access to the internet.

**Security Groups** – These act as firewall rules to control who can access my instances.

**EC2 Instances** – Virtual machines where I'd actually run my applications and services.

**Elastic IP** – A static public IP address so I could always reach my resources at the same address.

My goal was simple: build something secure, scalable, and easy to manage

[]()

1. Creating a VPC

I started by opening the AWS Management Console and navigating to the VPC service. From there, I clicked on "Create VPC" and filled in the details:

**Name:** MyVPC

**IPv4 CIDR:** 10.0.0.0/16

**IPv6 CIDR:** I left this as "None" since I didn't need it for now

After entering everything, I clicked "Create" and my VPC was ready to go.

**Why this matters:** The VPC essentially creates my own isolated network within AWS. The CIDR block (10.0.0.0/16) gives me a large IP address range, which means I can create plenty of subnets down the line without running out of space.

![image.png](attachment:b6829c06-5f4b-4c62-af2c-6306954738d1:image.png)<img width="1333" height="776" alt="image" src="https://github.com/user-attachments/assets/98e7245a-cb46-412a-8e23-16ae656467ea" />



1. Creating Subnets

Next, I navigated to the Subnets section and clicked "Create Subnet." I made sure to select the VPC I had just created.

I decided to create two subnets to keep things organized:

**Public Subnet:** 10.0.1.0/24 – This would host anything that needs to be accessible from the internet, like web servers.

**Private Subnet:** 10.0.2.0/24 – This would be for internal resources like databases and backend services that shouldn't be exposed directly to the outside world.

**Why this matters:** The public subnet can communicate with the internet, while the private subnet stays locked down internally. This separation adds an extra layer of security to my infrastructure.

![Screenshot 2026-02-21 142120.png](attachment:c23ac742-9bb8-4dd8-a23a-9bbcd6be0712:Screenshot_2026-02-21_142120.png)

1. Attaching an Internet Gateway

Now that I had my VPC and subnets set up, I needed a way for my public subnet to actually communicate with the internet. That's where the Internet Gateway comes in.

Here's what I did:

I went to the **Internet Gateways** section in the VPC dashboard and clicked **"Create Internet Gateway."** I gave it a simple name like **MyInternetGateway** and created it.

Once it was created, I selected it from the list and clicked **"Attach to VPC."** I chose **MyVPC** from the dropdown and confirmed the attachment.

**Why this matters:** The Internet Gateway is essentially the bridge between my VPC and the outside world. Without it, none of my resources in the public subnet would be able to send or receive traffic from the internet. Think of it like installing a front door to your house—without it, you can't get in or out. By attaching it to my VPC, I'm allowing any resources I place in the public subnet to communicate with the internet, which is essential for things like web servers or APIs that need to be publicly accessible.

![Screenshot 2026-02-21 142208.png](attachment:7276a734-cb78-4033-b16a-0921c5e5cd39:Screenshot_2026-02-21_142208.png)

1. Here's what I did:

I went to the **Route Tables** section in the VPC dashboard and clicked **"Create Route Table."** I gave it a name like **MyPublicRouteTable** and made sure it was associated with **MyVPC**.

Once it was created, I clicked on it and went to the **Subnet Associations** tab. I then associated it with my **Public Subnet** so that any traffic from that subnet would follow these routing rules.

Next, I added a route to actually connect to the internet:

- **Destination:** 0.0.0.0/0 (this means "anywhere on the internet")
- **Target:** My Internet Gateway

**Why this matters:** This route is basically saying, "Hey, if traffic from the public subnet wants to go anywhere on the internet, send it through the Internet Gateway." Without this route, even though the gateway is attached, my instances wouldn't know to use it. It's like having a door but no sign pointing to it—you need both for it to work.

I intentionally **did not** add this route to my Private Subnet's route table. That way, anything in the private subnet stays locked down and can't directly communicate with the internet, which keeps my sensitive resources like databases much more secure.

![Screenshot 2026-02-21 142120.png](attachment:c23ac742-9bb8-4dd8-a23a-9bbcd6be0712:Screenshot_2026-02-21_142120.png)

1. **Create flow log**

---

### Configure Flow Log

Fill in the following:

**Filter**

- `All` → capture all traffic
- `Accept` → accepted traffic only
- `Reject` → rejected traffic only

**Destination**

 . S3 bucket (you have create this during the process) and forget to copy you amason resource name (ASN). then copy it to the destination 

- Send logs to **Amazon CloudWatch** (most common)

**Log format**

- Choose **Default**

**IAM Role**

- Create or select an IAM role that allows VPC to send logs to CloudWatch.

---

### Create Flow Log

Click **Create flow log**.

Now AWS will start recording network traffic.

---

## What Flow Logs Show

Flow logs show information like:

- Source IP
- Destination IP
- Port number
- Protocol
- Accepted or rejected traffic

Example log record:

```
2 123456789 eni-abc123 10.0.1.10 10.0.2.5 443 52344 6 10 840 ACCEPT OK
```

1. NACLS:

## How to Add a NACL in AWS

1. Go to **VPC Dashboard**
2. Click **Network ACLs**
3. Click **Create network ACL**
4. Attach it to your **VPC**
5. Associate it with a **subnet**
6. Add **inbound and outbound rules**

1. Creating Security Groups

Now it was time to set up some security rules. Security groups in AWS work like a firewall—they decide who can talk to your resources and who can't.

I went to the **Security Groups** section and clicked **"Create Security Group."** I gave it a name: **MyWebSG**.

Then I configured the rules:

**Inbound rules** (who can connect to my server):

**SSH (port 22)** – I set this to allow only my IP address, so I could securely log in without letting the whole internet in.

**HTTP (port 80)** – I set this to "Anywhere" so that anyone could visit my website once it's up and running.

**Outbound rules** (what my server can connect to): I left this as "Allow all" so my server could freely reach out to download updates or communicate with other services.

**Why this matters:** Security groups are like bouncers at the door of your server. They decide who gets in and what can go out. By limiting SSH access to just my IP, I'm making sure no one else can try to break into my machine. And by allowing HTTP from anywhere, I'm making sure visitors can actually see my site.

![Screenshot 2026-02-21 142446.png](attachment:4319c6a0-7d41-4c38-af82-0538764bddf5:Screenshot_2026-02-21_142446.png)

1. **Launching an EC2 Instance**

Alright, this is where things get exciting—time to actually spin up my virtual server! I headed over to the EC2 dashboard and clicked that big orange **"Launch Instance"** button.

First up, I had to pick an **AMI (Amazon Machine Image)**—basically, what operating system I wanted my server to run. I went with **Amazon Linux 2** because it's lightweight and plays really well with AWS services, but if you're more of a Windows person, **Windows Server** is totally an option too.

Next came the **instance type**. I chose **t2.micro** because, hey, it's free tier eligible and perfect for testing things out without burning through cash. It's not the most powerful machine in the world, but for what I needed, it was more than enough.

Now for the fun part—configuring the network settings:

- I made sure to select **MyVPC** so everything stayed in my isolated network.
- I dropped it into the **Public Subnet** since I wanted this instance to be reachable from the internet.
- I toggled on **Auto-assign Public IP**—this gives my server a public IP address automatically so I can actually connect to it from my laptop.

For storage, I stuck with the default **8GB**. That's plenty for a basic server, and I can always add more later if I need to.

Then I attached the **MyWebSG** security group I created earlier—this is what controls who can talk to my server and keeps the bad guys out.

Finally, I hit **Launch**, and AWS prompted me to download a **key pair**. This is super important—it's basically the password I'll use to SSH into my server. I saved it somewhere safe because if I lose it, I'm locked out for good.

**Why this matters:** The EC2 instance is the heart of my cloud infrastructure. It's the actual virtual machine where I can run applications, host websites, or do pretty much anything I'd do on a physical server—except it's all in the cloud. The public IP means I can connect to it from anywhere, whether I'm SSH-ing in from my terminal (on Linux) or using Remote Desktop (on Windows). It's like having my own computer running 24/7 in Amazon's data center.

![Screenshot 2026-02-21 143022.png](attachment:3c063854-a1e8-4075-a86e-0827a9744791:Screenshot_2026-02-21_143022.png)

1. Assigning an Elastic IP (Optional but Recommended)

Once my EC2 instance was up and running, I noticed something: every time I stopped and restarted it, AWS would assign it a different public IP address. That's a problem if I want to reliably connect to my server or point a domain name to it.

So I decided to allocate an **Elastic IP**—a static public IP address that sticks with my instance no matter what.

Here's how I did it:

I went to the **Elastic IPs** section in the EC2 dashboard and clicked **"Allocate Elastic IP address."** AWS assigned me a new IP from their pool.

Then I selected that IP, clicked **"Actions,"** and chose **"Associate Elastic IP address."** I picked my EC2 instance from the list and confirmed.

**Why this matters:** Now, even if I stop my instance to save costs or reboot it for maintenance, the IP address stays the same. This is super useful if I'm hosting a website or running a service that people need to access consistently. Think of it like having a permanent phone number instead of getting a new one every time you restart your phone.

1. Connecting to My Virtual Machine

Alright, time to actually log into my server and start using it! The way you connect depends on what operating system your EC2 instance is running.

**If you're running Linux (like I was with Amazon Linux 2):**

I opened up my terminal and used SSH to connect. The command looked something like this:

```bash
ssh -i mykey.pem ec2-user@your-elastic-ip
```

Here's what's happening:

- `-i mykey.pem` tells SSH to use the private key I downloaded earlier when I launched the instance.
- `ec2-user` is the default username for Amazon Linux instances.
- `your-elastic-ip` is the public IP address (or Elastic IP) of my server.

Once I hit enter, I was in! I could now run commands, install software, and configure my server however I wanted.

**If you're running Windows Server:**

You'd use **Remote Desktop Protocol (RDP)** instead. You'd download an RDP file from the EC2 console, decrypt your admin password using your key pair, and then connect just like you would to any Windows machine remotely.

**Why this matters:** Being able to connect to your instance is crucial—it's how you actually manage and use your server. Whether you're installing a web server, setting up databases, or just poking around, SSH (or RDP) is your gateway in.

1. Testing Everything to Make Sure It Works

Before I called it a day, I wanted to make absolutely sure everything was set up correctly and securely. So I ran a few quick tests.

**First, I tested basic connectivity:**

I opened my terminal and pinged the public IP of my instance:

```bash
ping your-elastic-ip
```

If I got replies, that meant my instance was up and reachable from the internet. Success!

**Next, I tested HTTP access:**

If I had installed a web server (like Apache or Nginx), I'd open a browser and type in my instance's public IP. If I saw a webpage load, that meant my security group was correctly allowing HTTP traffic on port 80.

**Finally, I confirmed my private subnet was actually private:**

I tried to ping or access any resources I had placed in the private subnet from my local machine. As expected, I couldn't reach them directly from the internet—exactly what I wanted. This confirmed that my private subnet was locked down and only accessible internally within my VPC.

**Why this matters:** Testing is how you catch mistakes before they become problems. By verifying connectivity, web access, and subnet isolation, I made sure my infrastructure was not only functional but also secure. If something didn't work, I'd know right away and could troubleshoot before moving forward.

1. Wrapping It All Up

And that's it! I successfully built my own secure, scalable cloud infrastructure on AWS from scratch. It might have seemed like a lot of steps at first, but once you break it down, it's really just a matter of piecing together a few key components.

Here's a quick recap of what I set up:

- **VPC:** My own isolated network in the cloud
- **Subnets:** Public and private sections to separate what's exposed and what's hidden
- **Internet Gateway:** The bridge that lets my public resources talk to the internet
- **Security Groups:** Firewall rules that control who can access my resources
- **EC2 Instance:** My virtual machine where I can run applications and services
- **Elastic IP:** A permanent public address so I can always reach my server

Now I have a solid foundation that I can use to host websites, run applications, or experiment with cloud services—all while keeping things secure and under control.

The best part? This setup is flexible. I can scale it up, add more instances, attach databases, or even integrate other AWS services as my needs grow. The cloud is my playground now, and I'm just getting started.
