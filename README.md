# seniorcapstone
Progress of Senior Capstone that involves Nginx, Flask, Gunicorn, MySQL, Security Onion, and AWS.

Description:
As technology evolves at new exponential heights, security and protection of consumer and company data is at an all-time high. The purpose of the project is to become familiar with the fundamentals of setting up a virtual network with web and database servers, using enterprise level tools to make the virtual network secure and recoverable in case of a disaster, and analyzing and logging different malicious attacks of the network. This project will be using Security Onion as well as AWS Elastic Disaster Recovery tool and will be simulated on an Unraid 64 GB RAM server.

Applicaions and OS used:
Unraid OS, Ubuntu Server, Kali Desktop, Nginx, Flask, Gunicorn, MySQL, Security Onion, AWS

Midterm progress report:
This section outlines the progress made during Phase 1 of the project. The next few sections will be
discussing the setup of the Virtual Machines, servers, and the final results of Phase 1. They will include the step-by-step processes of setting up the servers, as well as briefly discussing any issues that were
encountered during set up and execution.

This project takes place on a custom server dedicated to education and homelabbing. This server runs on
UnraidOS, and is configured through the Unraid GUI and connected to via Tailscale. The server as of this point contains 4 VM instances, shown below: <br><br>

<img width="1109" height="199" alt="image" src="https://github.com/user-attachments/assets/87b2db1c-43e0-4305-8df1-c7d49bd83669" />
Figure 1.1: Shows Kali Desktop, SecurityOnion, MySQL Server (run on Ubuntu Server), and the Web Server (using Nginx and Flask, running on Ubuntu Server)<br><br>

Unraid automatically uses QEMU as an emulator, and VM configuration looks similar to the figure below:
 <img width="928" height="634" alt="image" src="https://github.com/user-attachments/assets/12ed855a-aff9-4e4f-8d4a-fb6dc3bef8ba" />

Figure 1.2: Show MySQL Server VM configuration as an example of what the VM configuration looks like.<br><br>

There are more configuration options, however it cannot fit on one screen. This part of the project took the longest, since I encountered errors with setting up the VM instances from the beginning. Various issues such as using the default CPU core, setting the initial memory as the max memory, and ensuring the boot order of the iso took priority were all things I needed to research to fix. 
Another issue encountered with Unraid, was that the default network option was br0. This normally would not be an issue, however I was not the only user on-site that had an Unraid server. Luckily, after conducting an nmap scan ensuring that all VM instances were detectable before starting the project, I realized I was able to see all containers from the other Unraid server that was on-site. I then switched the network to virtio-net to ensure isolation.
For future references, here is the nmap scan of the network for all IP addresses: <br>
 <img width="518" height="750" alt="image" src="https://github.com/user-attachments/assets/e7dd4e69-9d2e-46e0-8a54-78518c7bf208" /> <br>
Figure 1.3 Show the IP addresses and open ports for the web server and MySql server<br><br>

Setting up MySQL
The MySQL server was set up on Ubuntu Server OS. This was one of the more basic setups. After installing MySQL via sudo apt install mysql-server, it was time to create the database to contain any information gathered from the website. The information I decided to gather was a name the user could submit, and a email. 
After accessing MySQL via executing mysql -u root as the root user, I then used:
 CREATE DATABASE websitedatabase; 
 then used:
CREATE TABLE users (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(50), email VARCHAR(100));. 
I used AUTO_INCREMENT PRIMARY KEY to make it so that each row has a unique identifier. To ensure that the table was created, I used SHOW TABLES;
 <img width="655" height="269" alt="image" src="https://github.com/user-attachments/assets/fab37528-a29f-4c0d-96ea-826ec010c802" />

Figure 2.1 Shows users table in websitedatabase database<br><br>
After setting up Nginx and Flask on the web server, I learned that in order for flask to access the MySQL Database, I needed to create a special user for it. The command I used is as follows:
CREATE USER ‘flaskuser’@’192.168.122.25’ IDENTIFIED BY  ‘password’;
This means that I created a new user named flaskuser, and gave it the password “password”. I don’t particularly care about being too secure, since this creates another opportunity for penetration testing later on in the project. To verify that the new user was added, I used the prompt below:

 <img width="753" height="500" alt="image" src="https://github.com/user-attachments/assets/bcf387e2-c4d0-4a8c-bdc1-79f57c2459e0" />
 
 Figure 2.2 Shows that using SELECT user, host FROM mysql.user; shows all users with access to MySQL<br><br>

After using FLUSH PRIVILEGES; to give the new flaskuser access to the database (without rebooting), I exited MySQL and went to the /etc/mysql directory to edit the mysqld.conf file. Here, we needed to change the “bind-address = 127.0.0.1” to “bind-address = 0.0.0.0”. This allows remote access to the MySQL server from other machines (It is noted that this is NOT secure since anything could remote into it, but an easy option and a vulnerability to exploit for later).
We are now done with the MySQL Server setup.

Setting up the Webserver
The web server is hosted on Ubuntu Server and deliberately deployed on a separate virtual machine from the database server. Although this project focuses on penetration testing and Security Onion monitoring, I intentionally modelled the environment after a semi realistic on-prem enterprise network, where web and database servers are typically separated for security and performance.

After the initial set up of the Ubuntu server, I installed nginx via executing sudo apt install nginx. I then created a directory located at /var/www, called testwebsite. Here, I created the index.html file for the website, along with the CSS document for the website. Here is the contents of both documents below:
<img width="837" height="624" alt="image" src="https://github.com/user-attachments/assets/49f145cd-de5f-4ef5-a460-adac510cb8d5" />

Figure 3.1 Shows the content of the HTML file, most notably the form section that will be used later on<br><br>

 <img width="779" height="608" alt="image" src="https://github.com/user-attachments/assets/477d39a7-bd4c-4a4e-908d-071eb262edc8" />

Figure 3.2 Shows the basic content of the CSS file named styles.css <br><br>
After creating these documents, I realized that in order to connect the web server to the MySQL database, I needed another program. I decided to use Flask for this project. In short summary, Flask is a micro web framework written in Python that is used by major companies across the globe. For this instance Flask will help handle defining routes, returning responses and connecting and executing queries to MySQL database.
To do this, I needed to install python. After installing python, I needed to install Flask. This was a little tricky, since it needed to be installed through python’s virtual environment. After running virtualenv venv and source ./venv/bin/activate I am then able to install flask via pip install flask. After setup, we can exit the virtual environment with deactivate and make the Flask file for the web server. For simplicity, I included the file in the same directory as the index.html file and styles.css file which is located at /var/www/testwebsite. Below is the contents of the python file that will be utilizing flask:
<img width="795" height="546" alt="image" src="https://github.com/user-attachments/assets/d888adb6-6b0b-492f-a574-12b51732df58" />

Figure 3.3 Shows the contents of the file "flasktest.py"<br><br>
In the contents of the file, db_config highlights that the user, specifically flaskuser located at 192.168.122.218 (the MySQL server) will be the one conducting the execution of the queries later in the contents. 
In the def submit() section, it labels the inputs from the form made in the HTML file as name and email, then connects to the MySQL database through the configuration above, the flaskuser then executes the INSERT query for the obtained name and email, then closes the cursor and connection to the server. This then returns a different page to the user showing success.
To tie everything together, we also needed to install gunicorn. Gunicorn allows Flask to serve flasktest.py, as well as handle multiple requests via worker models. It is used for security and loadbalancing between Nginx and Flask. Gunicorn is also installed in the virtual environment for python via pip install gunicorn.
Finally, we need to configure Nginx so it serves the index.html file, states how many requests it can take and will proxy requests to Flask via Gunicorn. Below is the edited configuration file located at /etc/nginx:
 <img width="755" height="567" alt="image" src="https://github.com/user-attachments/assets/0b0b0231-812f-4df4-9883-451b26fb6b59" />

Figure 3.4 This shows the contents of the nginx.conf file<br><br>
Now Nginx will need to be reloaded with systemctl reload nginx with root user.

The Final Result of Phase 1
In order to start the webserver to receive requests with Gunicorn, I need to navigate to /var/www/testwebsite and start the python virtual environment there and run Gunicorn as shown below:
 <img width="1009" height="214" alt="image" src="https://github.com/user-attachments/assets/d8001a6d-52c7-4b00-86ba-c5c21e908941" />

Figure 4.1 Shows the virutal environment activation and gunicorn command<br><br>
This command means that there are two worker models handling requests to send to flasktest.py, which can then be run and executed. 
Switching to the Kali VM, we can see on the network (if we put in the web server IP, which is 192.168.122.25) we will receive this webpage:
 <img width="474" height="696" alt="image" src="https://github.com/user-attachments/assets/06208499-693b-444b-a74b-15a0007d125c" />

Figure 4.2 Shows index.html, along with examples filled out in the Name and email text boxes<br><br>
For some reason, the CSS document is not executing and is still an issue currently. However, when we insert a name and email for the page and submit it by clicking the button, the user will receive this response:
 <img width="422" height="561" alt="image" src="https://github.com/user-attachments/assets/aba882b9-1581-45e7-aa83-e0e7c72e35ba" />

Figure 2.3 The response the user recieves after clicking the "GG EZ" button<br><br>

Now to see if Flask inserted our data into the MySQL database, we transition to the MySQL Server VM, login through root, execute USE websitedatabase; and use SELECT * FROM users;
The output of the table is shown below:
 <img width="661" height="495" alt="image" src="https://github.com/user-attachments/assets/880d9970-539f-4404-8066-c2d1f3153268" />

Figure 3.4 Here is the information taken from the website and inserted into the users table<br><br>

This now concludes the final part of Phase One, and setting up the foundations of a simulated network. Moving forward, the project will transition into Phase Two, focusing on penetration testing and the configuration and utilization of Security Onion. Following this phase, the work will progress toward integrating AWS Elastic Disaster Recovery to simulate cloud-based recovery.
