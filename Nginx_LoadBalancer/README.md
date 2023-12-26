# **Nginx and LoadBalancing**
![nginx](./img/nginx_image.png)

Load balancing is an excellent way to scale out an application and increase its performance and redundancy. Nginx, a popular web server software, can be configured as a simple yet powerful load balancer to improve your server’s resource availability and efficiency.
*Nginx acts as a single entry point to a distributed web application working on multiple separate servers.*
![nginx](./img/LB_diagram.png)
## *Installing Nginx*
The first thing to do is to set up *two servers* on a cloud like AWS.
![ssh](./img/webservers_instantiation.png)
![ssh](./img/instances_running.png) In this project, i have used an *Ubuntu Linux distribution*
![ssh](./img/ssh_webserver1.png)
![ssh](./img/ssh_webserver2.png)

We then install apache2 web servers on the two instances
`sudo apt update -y`
`sudo apt install apache2 -y`
![apache](./img/apache2_install_automation.png)
*This can either be done automatically or usage of the code blocks above*. I chose to automate it here in AWS

The next is to set up an instance for loadbalancing
![server](./img/instances_running.png)
`sudo apt update -y`
`sudo apt install nginx -y`

Then we check if our apps are running and active `sudo systemctl status nginx`
![webnginx](./img/nginx_loadbalancer_installed.png)
`sudo systemctl status apache2`
![apache2](./img/apache2_running_web1.png)
![apache2](./img/apache2_running_web2.png)


I then added a new listening directive for port 8000
![port](./img/vi_config.png)
![listen](./img/vi_config_ips_port.png)
![resrat](./img/loadbalancer_config.png)

An *index.html* file should be created, where the server will serve it's content from
![index](./img/index_web01.png)

Change file Ownership of the index.html thus 
`sudo chown www-data:www-data ./index.html`

## Nginx as Loadbalancer

- On our Nginx server, we alter the configuration file thus
`sudo vi /etc/nginx/conf.d/loadbalancer.conf`
![conf](./img/loadbalancer_config.png)

- secondly test the configuration
`ln`
![ln](./img/link_test.png)

- we thereafter restart nginx
`sudo systemctl restart nginx`
![web](./img/nginx_reload.png)
- check what is being served
`curl localhost`
![web](./img/vi_reload_curllocalhost.png)
### **Paste the public ip of your nginx instance in your http browser**

![web](./img/load_web01.png)
![web](./img/load_web02.png)
![web](./img/voila.png)
### **Conclusion**
A load balancer acts as the “traffic cop” sitting in front of your servers and routing client requests across all servers capable of fulfilling those requests in a manner that maximizes speed and capacity utilization and ensures that no one server is overworked, which could degrade performance. If a single server goes down, the load balancer redirects traffic to the remaining online servers. When a new server is added to the server group, the load balancer automatically starts to send requests to it.

