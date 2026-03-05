# seniorcapstone
Progress of Senior Capstone that involves Nginx, Flask, Gunicorn, MySQL, Security Onion, and AWS.

<h1>Executive Summary</h1>
As technology continues to evolve at an unprecedented rate, the need for robust
cybersecurity infrastructure is at an all-time high. Organizations face increasing threats to
their networks, data, and operations, emphasizing the importance of proactive defense
and disaster recovery strategies.<br><br>
This capstone project focuses on the design, configuration, and analysis of a secure virtual
network using enterprise-level tools. By integrating Security Onion for intrusion detection
and AWS Elastic Disaster Recovery for cloud-based resilience, the project aims to simulate
a real-world enterprise environment that can both detect and recover from cyberattacks.
The implementation will take place within a virtualized Unraid server environment,
providing a realistic and scalable testing ground for monitoring, analysis, and recovery.
Ultimately, this project will enhance technical competencies in network security
management, cloud disaster recovery, and penetration testing while contributing to a
deeper understanding of the cybersecurity lifecycle.<br><br>
<h2>Projected Milestone</h2>
The projected milestone for this project is the set up of a virtual on-prem network utilizing
a web server and a database server, the usage of Security Onion to log and monitor traffic,
and the usage of AWS for disaster recovery simulation.<br><br>
<h2>Deliverables Achieved</h2>
<ul>
These are the deliverables that were achieved throughout the project:
<li>Setting up a web server (Nginx) and a database server (MySQL)</li>
<li>Connecting the database server and web server through Flask</li>
<li>Setting Up Security Onion and logging events</li>
<li>Setting up AWS and replicating the MySQL server</li>
</ul>

<h1>Phase 1</h1>
This section outlines the progress made during Phase 1 of the project. The next few
sections will be discussing the setup of the Virtual Machines, servers, and the final results
of Phase 1. They will include the step-by-step processes of setting up the servers, as well
as briefly discussing any issues that were encountered during set up and execution.
<h2>Setting up VM's</h2>
This project takes place on a custom server dedicated to education and homelabbing. This server runs on
UnraidOS, and is configured through the Unraid GUI and connected to via Tailscale. The server as of this point contains 4 VM instances, shown below: <br><br>

<img width="1109" height="199" alt="image" src="https://github.com/user-attachments/assets/87b2db1c-43e0-4305-8df1-c7d49bd83669" />

_Figure 1.1: Shows Kali Desktop, SecurityOnion, MySQL Server (run on Ubuntu Server), and the Web Server (using Nginx and Flask, running on Ubuntu Server)_ <br><br>

Unraid automatically uses QEMU as an emulator, and VM configuration looks similar to the figure below:
 <img width="928" height="634" alt="image" src="https://github.com/user-attachments/assets/12ed855a-aff9-4e4f-8d4a-fb6dc3bef8ba" />

_Figure 1.2: Show MySQL Server VM configuration as an example of what the VM configuration looks like._<br><br>

There are more configuration options, however it cannot fit on one screen. This part of the project took the longest, since I encountered errors with setting up the VM instances from the beginning. Various issues such as using the default CPU core, setting the initial memory as the max memory, and ensuring the boot order of the iso took priority were all things I needed to research to fix. 
Another issue encountered with Unraid, was that the default network option was br0. This normally would not be an issue, however I was not the only user on-site that had an Unraid server. Luckily, after conducting an nmap scan ensuring that all VM instances were detectable before starting the project, I realized I was able to see all containers from the other Unraid server that was on-site. I then switched the network to virtio-net to ensure isolation.<br><br>
For future references, here is the nmap scan of the network for all IP addresses: <br>
 <img width="518" height="750" alt="image" src="https://github.com/user-attachments/assets/e7dd4e69-9d2e-46e0-8a54-78518c7bf208" /> <br>
_Figure 1.3 Show the IP addresses and open ports for the web server and MySql server_<br><br>

<h2>Setting up MySQL</h2>
The MySQL server was set up on Ubuntu Server OS. This was one of the more basic setups. After installing MySQL via **sudo apt install mysql-server**, it was time to create the database to contain any information gathered from the website. The information I decided to gather was a name the user could submit, and a email. 
After accessing MySQL via executing **mysql -u root** as the root user, I then used:
 **CREATE DATABASE websitedatabase**; 
 then used:
**CREATE TABLE users (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(50), email VARCHAR(100));**. 
I used AUTO_INCREMENT PRIMARY KEY to make it so that each row has a unique identifier. To ensure that the table was created, I used SHOW TABLES;
 <img width="655" height="269" alt="image" src="https://github.com/user-attachments/assets/fab37528-a29f-4c0d-96ea-826ec010c802" />

_Figure 2.1 Shows users table in websitedatabase database_<br><br>

After setting up Nginx and Flask on the web server, I learned that in order for flask to access the MySQL Database, I needed to create a special user for it. The command I used is as follows:
CREATE USER ‘flaskuser’@’192.168.122.25’ IDENTIFIED BY  ‘password’;
This means that I created a new user named flaskuser, and gave it the password “password”. I don’t particularly care about being too secure, since this creates another opportunity for penetration testing later on in the project. To verify that the new user was added, I used the prompt below:

 <img width="753" height="500" alt="image" src="https://github.com/user-attachments/assets/bcf387e2-c4d0-4a8c-bdc1-79f57c2459e0" />
 
_Figure 2.2 Shows that using SELECT user, host FROM mysql.user; shows all users with access to MySQL_<br><br>

After using FLUSH PRIVILEGES; to give the new flaskuser access to the database (without rebooting), I exited MySQL and went to the /etc/mysql directory to edit the mysqld.conf file. Here, we needed to change the “bind-address = 127.0.0.1” to “bind-address = 0.0.0.0”. This allows remote access to the MySQL server from other machines (It is noted that this is NOT secure since anything could remote into it, but an easy option and a vulnerability to exploit for later).
We are now done with the MySQL Server setup.

<h2>Setting up the Webserver</h2>
The web server is hosted on Ubuntu Server and deliberately deployed on a separate virtual machine from the database server. Although this project focuses on penetration testing and Security Onion monitoring, I intentionally modelled the environment after a semi realistic on-prem enterprise network, where web and database servers are typically separated for security and performance.

After the initial set up of the Ubuntu server, I installed nginx via executing sudo apt install nginx. I then created a directory located at /var/www, called testwebsite. Here, I created the index.html file for the website, along with the CSS document for the website. Here is the contents of both documents below:
<img width="837" height="624" alt="image" src="https://github.com/user-attachments/assets/49f145cd-de5f-4ef5-a460-adac510cb8d5" />

_Figure 3.1 Shows the content of the HTML file, most notably the form section that will be used later on_<br><br>

 <img width="779" height="608" alt="image" src="https://github.com/user-attachments/assets/477d39a7-bd4c-4a4e-908d-071eb262edc8" />

_Figure 3.2 Shows the basic content of the CSS file named styles.css_ <br><br>

After creating these documents, I realized that in order to connect the web server to the MySQL database, I needed another program. I decided to use Flask for this project. In short summary, Flask is a micro web framework written in Python that is used by major companies across the globe. For this instance Flask will help handle defining routes, returning responses and connecting and executing queries to MySQL database.<br><br>
To do this, I needed to install python. After installing python, I needed to install Flask. This was a little tricky, since it needed to be installed through python’s virtual environment. After running virtualenv venv and source ./venv/bin/activate I am then able to install flask via pip install flask. After setup, we can exit the virtual environment with deactivate and make the Flask file for the web server. For simplicity, I included the file in the same directory as the index.html file and styles.css file which is located at /var/www/testwebsite. Below is the contents of the python file that will be utilizing flask:
<img width="795" height="546" alt="image" src="https://github.com/user-attachments/assets/d888adb6-6b0b-492f-a574-12b51732df58" />

_Figure 3.3 Shows the contents of the file "flasktest.py"_ <br><br>

In the contents of the file, db_config highlights that the user, specifically flaskuser located at 192.168.122.218 (the MySQL server) will be the one conducting the execution of the queries later in the contents. 
In the def submit() section, it labels the inputs from the form made in the HTML file as name and email, then connects to the MySQL database through the configuration above, the flaskuser then executes the INSERT query for the obtained name and email, then closes the cursor and connection to the server. This then returns a different page to the user showing success.<br><br>
To tie everything together, we also needed to install gunicorn. Gunicorn allows Flask to serve flasktest.py, as well as handle multiple requests via worker models. It is used for security and loadbalancing between Nginx and Flask. Gunicorn is also installed in the virtual environment for python via pip install gunicorn.<br><br>
Finally, we need to configure Nginx so it serves the index.html file, states how many requests it can take and will proxy requests to Flask via Gunicorn. Below is the edited configuration file located at /etc/nginx:
 <img width="755" height="567" alt="image" src="https://github.com/user-attachments/assets/0b0b0231-812f-4df4-9883-451b26fb6b59" />

_Figure 3.4 This shows the contents of the nginx.conf file_<br><br>

Now Nginx will need to be reloaded with systemctl reload nginx with root user.

<h2>The Final Result of Phase 1<h2></h2>
In order to start the webserver to receive requests with Gunicorn, I need to navigate to /var/www/testwebsite and start the python virtual environment there and run Gunicorn as shown below:
 <img width="1009" height="214" alt="image" src="https://github.com/user-attachments/assets/d8001a6d-52c7-4b00-86ba-c5c21e908941" />

_Figure 4.1 Shows the virutal environment activation and gunicorn command_<br><br>

This command means that there are two worker models handling requests to send to flasktest.py, which can then be run and executed. 
Switching to the Kali VM, we can see on the network (if we put in the web server IP, which is 192.168.122.25) we will receive this webpage:
 
 <img width="474" height="696" alt="image" src="https://github.com/user-attachments/assets/06208499-693b-444b-a74b-15a0007d125c" />

_Figure 4.2 Shows index.html, along with examples filled out in the Name and email text boxes_<br><br>

For some reason, the CSS document is not executing and is still an issue currently. However, when we insert a name and email for the page and submit it by clicking the button, the user will receive this response:
 
 <img width="422" height="561" alt="image" src="https://github.com/user-attachments/assets/aba882b9-1581-45e7-aa83-e0e7c72e35ba" />

_Figure 4.3 The response the user recieves after clicking the "GG EZ" button_<br><br>

Now to see if Flask inserted our data into the MySQL database, we transition to the MySQL Server VM, login through root, execute USE websitedatabase; and use SELECT * FROM users;
The output of the table is shown below:
 <img width="661" height="495" alt="image" src="https://github.com/user-attachments/assets/880d9970-539f-4404-8066-c2d1f3153268" />

_Figure 4.4 Here is the information taken from the website and inserted into the users table_<br><br>

This now concludes the final part of Phase One, and setting up the foundations of a simulated network. Moving forward, the project will transition into Phase Two, focusing on penetration testing and the configuration and utilization of Security Onion. Following this phase, the work will progress toward integrating AWS Elastic Disaster Recovery to simulate cloud-based recovery.

<h1>Phase 2</h1>

Phase 2 consists of the set-up of Security Onion, testing, and analysis of logs through
Security Onion web dashboard.<br><br>

<h2>Setting up Security Onion</h2><br>

The initial setting up of Security Onion was slightly difficult due to the lack of familarity
with the set up of Vms. The first issue that was encountered was the iso had been
corrupted during the download or upload process. A re-upload of the iso was able to fix
the problem.<br>

Here the set up process included the selection of Analyst role for Security Onion, setting
Security Onion to 12 GB of RAM (which allowed me to run all future Vms together on 32 GB
of server RAM).
<img width="1919" height="1194" alt="image" src="https://github.com/user-attachments/assets/20944653-fb5e-4c42-9aa3-835a992c7c61" />

_Figure 5.1: Here is the selection of NIC to chose from during the set up process_<br><br>

While entering the initial set-up, Security Onion requires a minimum of two NICs. This was
difficult, as the documentation of setup for Vms in Unraid is different compared to the
exorbitant documentation of Vmware or Virtual Box. After some research and trial and
error, two NICs were finally applied.<br>

The final set up for Security Onion is shown below:
<img width="1004" height="770" alt="image" src="https://github.com/user-attachments/assets/8936127b-113a-416a-bba7-168499c96014" />
_Figure 5.2: This is the final screen for Security Onion setup_<br><br>

Now we are able to access the terminal of Security Onion, shown below:
<img width="971" height="1142" alt="image" src="https://github.com/user-attachments/assets/945d5f97-d932-4bb3-968e-5438a5504996" />

_Figure 5.3: Logging into Security Onion for the first time_<br><br>

In order to access the web dashboard to Security Onion, we will need a trusted
computer/IP to monitor Security Onion activity.<br><br>

<h2>Setting up Analyst Computer for Security Onion</h2><br>
Here, we will set up a trusted computer to analyze traffic through the Security Onion
dashboard. This will be an Ubuntu desktop, with 4 GB RAM and 20 GB of physical memory
dedicated to the VM.
<img width="965" height="1137" alt="image" src="https://github.com/user-attachments/assets/084cbe64-fca0-49a2-b729-94dfb9462c45" />

_Figure 6.1: The set up for Ubuntu desktop_<br><br>

Now that we have set up the Ubuntu desktop, we can attempt to connect to the
dashboard. However, we will get this as a response:

<img width="976" height="1148" alt="image" src="https://github.com/user-attachments/assets/de786936-c832-474b-8642-196aaa42b5b2" />

_Figure 6.2: Rejected connection to web dashboard_<br><br>

This because by default, Security Onion blocks all access to the web dashboard. Do fix this,
we need to allow the admin computer access by whitelisting the IP shown below:
<img width="977" height="1145" alt="image" src="https://github.com/user-attachments/assets/0734a881-9309-4cc0-8d4c-8c88f480b154" />
_Figure 6.3: Allow IP to access web dashboard_<br><br>

By allowing our admin computer, 192.168.22.166, through the firewall of Security Onion
through so-firewall Now, we are able to access the dashboard:
<img width="1917" height="1199" alt="image" src="https://github.com/user-attachments/assets/64d1d14d-3096-4fe1-92fa-eaa68eda769b" />

_Figure 6.4: Login page for Security Onion webpage_<br><br>

Here, we use the email and password from Security Onion from initial setup.
After logging in, we are greeted with the Getting Started page:
<img width="1697" height="1061" alt="image" src="https://github.com/user-attachments/assets/ca736959-a02e-4406-a0cd-b80abf955f57" />

_Figure 6.5: Successful login to dashboard_<br><br>

Security Onion provides the activity of anything that passes through it by, logs it, and
monitors traffic at all times. In order to verify that it is properly working, we will need to
test it.<br><br>
An SQL injection is one of the more easier attacks to do on the network, and interestingly
enough, the most ineffective against the website. This is due to the usage of Flask, where
it inherently is hardened against injections by strict parameterized queries and a asolid
framework. The attack should not succeed, however Security Onion should still be able to
detect it. Here, will will go back to the Kali VM and access the website through there.
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/97f060cc-c5d5-4af7-bf4f-051cec425680" />

_Figure 6.6: Fixed website with SQL injection_<br><br>

Here the “attacker” will attempt to display multiple tables on the SQL database (which will
not work). We can submit the executable through clicking the button.<br>
If we return to the console of Security Onion, we can see that the malicious activity in the
alerts section of the dashboard:
<img width="1715" height="1072" alt="image" src="https://github.com/user-attachments/assets/6348cb10-f982-45a9-a10f-6671dc664f9d" />

_Figure 6.7: Security Onion Alert showing in-depth details of the severity of the attack_<br><br>

Here is displays a plethora of information, including the source and destination ports and
ips, which service was triggered, and the treat level of what occurred. Here in this example
it allowed the activity to pass, however it was marked as a medium level threat and
logged.<br>
This is the end of Phase 2.

<h1>Phase 3</h1><br>
The final phase will include the usage of AWS to simulate the uploading and downloading
of the MySQL database server. This will consist of replicating the database server, the set
up of EC2 environment in AWS, and the testing and process of the recovery process in
AWS.<br><br>
The set-up of this includes the assignment of various roles, permissions, creation of
security groups, creation of VPC, and the implementation of EC2 and replication of the
server. For the sake of this paper, some unnecessary and incorrect processes will be
omitted as this section will cover a lot of material.<br>

<h2>Setting up AWS for the first time</h2>
In order to start with the disaster recovery process, we must first crete an AWS account.
Once created and logged in, we are greeted with this screen:
<img width="1856" height="1160" alt="image" src="https://github.com/user-attachments/assets/08d60e11-cdd2-4cb3-bcf3-78d6697d2eff" />

_Figure 7.1: AWS dashboard after first time logging in_<br><br>

Here, we can navigate to the “AWS Elastic Disaster Recovery” page through the search
engine. Here, it will direct you towards the AWS wiki for more information setting up
Elastic Recovery:
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/cf5e7c72-47a0-4036-a532-3e4b89d22380" />

_Figure 7.2: AWS Elastic Disaster Recovery page_<br><br>

To start off, we will need to create a user that will have permissions to replicate the MySQL
server and generate the keys for this process. Here, we created a made-up user named
bigjack:
<img width="1828" height="1143" alt="image" src="https://github.com/user-attachments/assets/47ad980a-ac65-49f5-92a7-384c2000577b" />

_Figure 7.3: Creating a user to set permissions and roles_<br><br>

Bigjack will also need added permissions and roles to be able to properly complete tasks
later on. We will also need this user later for generating keys to connect the MySQL
replication agent and the AWS network we create in the future.<br>
Now, we need to create a VPC and security group for the network. Below is the current
VPC and security group that is being used (ones that re selected):
<img width="1850" height="1156" alt="image" src="https://github.com/user-attachments/assets/44b745ba-ee51-41b0-a9cf-a635228475e6" />

_Figure 7.4: This is the VPC (Virtual Private Cloud) dashboard_<br><br>

After the creation of the VPC and security groups, it was time to generate the keys needed
to replicate the server through the user bigjack.

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/199f24c8-dd1e-4f55-9444-a12272229c1a" />

_Figure 7.5: Here is where we will create access keys for replication_<br><br>

Now, we move to the MySQL server to begin installing the replication agent. For this
project, we will be setting up the replication on us-east-1:

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/b25d20b9-d714-426b-8992-4ad3cd98715a" />

_Figure 7.6: The replication agent installation process_<br><br>

Once the replication agent is installed, we can then move forward with connecting the
AWS user and network to the agent for replication. This is where we will use the keys
generated from the user:

<img width="1185" height="129" alt="image" src="https://github.com/user-attachments/assets/af94457a-d10d-4df0-bde8-64cb7e6fda69" />

_Figure 7.7: Giving the access keys and region where the VPC is_<br><br>

If the user has valid permissions (there are examples below where permissions were not
valid or invalid login attempt), the result will be similar to this:
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/04b99103-fbcc-4fa7-8c36-c0f5f9dcd9de" />

_Figure 7.8: Here are numerous attempts to replicate before a final successful replication_<br><br>

Now, we can go to the EC2 section of AWS, where we will find out server beginning to
replicate:

<img width="1703" height="1064" alt="image" src="https://github.com/user-attachments/assets/1a6fc132-4502-4284-9e44-77e44faf0ec2" />

_Figure 7.9: Here we can see the pending replication process in EC2_<br><br>

If we go to the AWS Disaster Recovery, we can also see the MySQL server in the source
server section:
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/92c23e84-9c2a-4124-bc2c-3e8630e959e6" />

_Figure 7.10: In-depth information on the instance_<br><br>

Here you can also look at important server information can be located, especially
replication and launch settings. Any errors encountered around this area can most likely
be fixed here by double checking the CIDR, VPC, and security group that this current
instance is using.<br><br>
After completing various validations (mainly correct usage of the groups of interest stated
above), we will finally reach the actual replication progress here:

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/b0d6ebb6-7389-4c0f-a2d8-95655e884380" />

_Figure 7.11: This is after various checklist items have been completed from AWS_ <br><br>

After waiting for the replication to complete, the EC2/Diaster Recovery instance will begin
to create a snapshot of itself.
<img width="1609" height="220" alt="image" src="https://github.com/user-attachments/assets/fb9ff7e8-276c-4c8a-a9ab-b164915ed2bf" />

_Figure 7.12: Creation of the snapshot of the instance_ <br><br>
Before attempting to run a recovery process, the user is able to initiate a drill process, to
test the implementation of a disaster recovery. It is highly recommended that one initiates
a drill process, as the criteria for the implementation of the recovery instance is different
than the replication process of an instance. An example of an error encountered is shown
below:
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/dcdfd55f-4f00-465c-81ce-7787e166af96" />

_Figure 7.13: Example error of the recovery drill failing due to kepping the private ip of the on-prem network server_ <br><br>

This error was caused by keeping the private IP of the server from the on-prem network to
the EC2 network that was set up. This can be changed in launch settings and removing the
IP, and allowing AWS to automatically assign an IP for the instance on the VPC network
that was set up.<br><br>
A successful drill recovery will look similar to the figure below:
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/e516b6c4-6130-4245-ad53-f0cdf5046936" />

_Figure 7.14: Successful recovery drill_ <br><br>

**THIS IS A DANGEROUS PROCESS. IF YOU’RE NETWORK IS NOT SECURE YOU WILL
BECOME VULNERABLE TO ATTACKERS.**

<img width="933" height="751" alt="image" src="https://github.com/user-attachments/assets/f7aca0b2-4a00-4ca1-95f8-6db1d1293cf6" />

_Figure 7.15: Attacker checking storage, users, and files of the server_<br><br>

Anything past the ^C was not me, but rather a unwelcome guest that had remoted into
the MySQL server. They obviously found nothing of interest, however this reinforced the
idea of always thinking about safety and ways to harden you networks and systems,
especially if it is just a project.

<img width="2100" height="1575" alt="image" src="https://github.com/user-attachments/assets/258dd369-febc-4d8e-b236-de7cb240d472" />
The server was shut down immediately after this, where upon I encountered a kernel
panic of a corrupted system config file and was unable to continue forward with the
project with the current timeline.

<h1>Conclusion and Reflection</h1>
This capstone project was able to successfully simulate the deployment and monitoring of
a on-prem network. This was done by creating a functional webpage through Nginx and
Flask, while also connecting a database server that used MySQL. Then Security Onion was
able to monitor network traffic, and alert the user of any suspicious activities. The project
was also able to incorporate the usage of AWS for recovery of servers.<br><br>
<ul>
 
</ul>
<li>Phase 1 established the foundation of the project as a whole. A web server with Ubuntu
was deployed, along with it serving basic HTML and CSS, and a separate VM server was
created to host MySQL. The two separate servers were connected through Flask to
simulate a simple segmentation in the network.</li>

<li>Phase 2 expanded the network slightly by using Security Onion to monitor and log traffic
that occurred through the network. Using the foundations set up in Phase 1, it created an
opportunity to simulate a SQL injection, which was successfully detected and logged by
Security Onion.</li>

<li>Phase 3 explored basic disaster recovery principles by setting up Virtual Private Clouds,
security groups, replication agents, and more. While initiating the drill lead to
unauthorized user accessing the MySQL database, it reinforced the idea of how critical it is
for learning environments to be secure, as well as instilling basic security principles.</li>
<br>
Despite all of the challenges encountered throughout the project, it was able to achieve its
core principles. It provided hands on experience of simulating a small network as well as
providing experience with working AWS. A goal that was not achieved was the ability to
monitor incoming intrusion through AWS through drill recovery. This was due to the
kernel panic, Security Onion missing ElasticSearch shards, and the replication agent on the
MySQL database breaking near the end of the project. This project servers as a personal
accomplishment with entering some of the basics of networking, and a present reminder
of the cybersecurity landscape and the importance of a resilient network.
