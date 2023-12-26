# Automating Loadbalancer configuration with Shell scripting

## Automating the Deployment of Webservers

In this module we will be automating the entire process of loadbalancing using shell scripting
![auto](./img/auto_img.png)
1. We need not provision two Ec2 instances
![ins](./img/2webservers.png)
![inst](./img/Instance_creation.png)
2. Edit Inbound rules to open up port 8000
![port](./img/port8000SG.png)
3. Next is to connect to our instances using ssh
![ssh](./img/ssh1.png)
![ssh](./img/ssh2.png)
4. Create a shell script
`vi install.sh`
and then write the script with a placeholder for our PublicIP
![script](./img/vi_instal01.png)
![script](./img/placeholder.png)
5. Make the script executable `chmod u+x install.sh`
![x](./img/chmod.png)
6. Run the script thus `./install.sh PUBLICIP`
![p](./img/parse_publicIP.png)
7. confirm if Apache2 is running on our webservers
![apache2](./img/running1.png)![apache2](./img/running2.png)

## Deploying and configuring Nginx as a LoadBalancer using Shell scripting
To start with we will ssh into the instance of our loadbalancer.
![ldb](./img/ldb_instance1.png)
![ldb](./img/ldb_instance2.png)
We write a script that ![ng](./img/ng2_.png)
1. update our machine and install nginx ![ng](./img/ng1.png)
2. creates the *loadbalancer config file* 
3. receives the *publicIP of the two webservers and the nginx as an argument* and uses the IP of two webservers as backend servers![ng](./img/ng2.png)
4. this scripts also tells our server to listen to port80![ng](./img/ng3.png)
5. checks if the config is ok ![ng](./img/ng4.png)
6. restarts nginx ![ng](./img/ng5.png)
7. we need to make this file executable `sudo chmod +x nginx.sh`![ng](./img/exe.png)
8. we then need to parse the *PUBLICIPS AS ARGUMENTS* thus `./nginx.sh PUBLIC_IP Webserver-1 Webserver-2`
where the variables are
- publicIP of the loadbalancer
- and the webservers respectively
![run](./img/run_script.png) ![run](./img/end.png)

we can check if our config file is present

![run](./img/config_file.png)

### *Paste the publicIP of the loadbalancer*
![Alt text](./img/web1.png)![Alt text](./img/web2.png)

## **Conclusion**

A shell script is a text file that contains a sequence of commands for a UNIX-based operating system. It is called a shell script because it combines a sequence of commands, that would otherwise have to be typed into the keyboard one at a time, into a single script.This is very useful in deploying servers really fast![Alt text](./img/img.png)