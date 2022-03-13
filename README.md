

# How to install nginx loadbalancer on amzon-linux ec2-instance
-------------------------------------------------- 


In this article, I will explain how to install and configure Ngix on ec2 amzon-linux

The process for installing & configuring Nginx on RHEL, Centos and Amazone linx is the same. 

![](https://i.ibb.co/b795zsH/Untitled-Diagram-drawio-1.png)


# Master Instance setup
-------------------------------------------------- 

Launch an amzon-linux instance using the management console. While launching the instance, configure the security group to allow traffic from HTTP 80 port & HTTPS 443


# Install Nginx on amzon-linux
-------------------------------------------------- 

#### Step1: In amazon-linux you have to set up the epel(extra packages for enterprise Linux) repo to install Nginx. To install EPEL package repository, follow the steps below or you can use the amazon-linux-extras for enabling the EPEL repo and then install Nginx.

```javascript
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```

#### Step2: Execute the following.

```
sudo yum install -y epel-release
```

#### Step3: Update the packages list

```
sudo yum update -y
```

#### Install Nginx

```
sudo yum install nginx -y
```

# Start and Enable Nginx
-------------------------------------------------- 

#### Step 1 : Check the verstion to make sure Nginx is installed

```
sudo nginx -V
```


#### Step 2: Start and enable Nginx

```
sudo systemctl start nginx
sudo systemctl enable nginx
```

#### Step3: Check the status Nginx to make sure it is running as expected.

```
sudo systemctl status nginx
```

#### Step4: Visit the nginx page using the Server IP. You should be seeing 

```
http://[server-IP]
```



# Client Instance setup
-------------------------------------------------- 

Launch two amzon-linux instance using the management console. While launching the instance, configure the security group to allow traffic from HTTP 80 port & HTTPS 443

Please follow the same step we have done to [install Nginx](#Install-Nginx-on-amzon-linux) on the master server and then [start and enable nginx](#Start-and-Enable-Nginx) on Client machine

When you have one or more website to be hosted on the same Nginx server, you need to use the Virtual Hosts configration.


In this section, I will show you how to create a virtual host configration for a website.

#### Step 1: Create the website forlder and a public folder to put the static assets inside /var/www

> Here am going to give the name as domain.com. Replace the name with your website name. 

```
sudo mkdir -p /var/www/domain.com/public_html
```

#### Step 2: Create a test index.html file if you dont have your own index file.

```
sudo vi /var/www/domain.com/public_html/index.html
```

> Copy the below content to /var/www/domain.com/public_html/index.html
```
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8">
    <title>Welcome to domian.com</title>
  </head>
  <body>
    <h1>Welcome To domain.com home page!</h1>
  </body>
</html>

```
#### Step3: Change the ownership of the root document to the use which is managing the worker process.

```
sudo chown -R nginx.nginx /var/www/domain.com
```

#### Step4 : Create a Nginx configration file with the website name

```
sudo vi /etc/nginx/conf.d/domain.com.conf
```

> Copy the following server block configration the conf file (/etc/nginx/conf.d/domain.com.conf) and save it



### **_Please note : Replace the domain.com with your domain name._**

```
server {
    listen 80;
    listen [::]:80;

    root /var/www/domain.com/public_html;

    index index.html;

    server_name domain.com www.domain.com;

    access_log /var/log/nginx/domain.com.access.log;
    error_log /var/log/nginx/domain.com.error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
#### Step5: Validate the configration file using the following command

```
sudo nginx -t
```

> If should get the following success message. If no, please check the configration file.

```
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

#### Step6: Restart the nginx server.

```
sudo systemctl restart nginx
```


#### Do the same steps that we did on the first amazon instance to the second client amazon instance [Click here for more infomation](#Client-Instance-setup).

### **_Note that please replace the domain with your domain name when you setup the Nginx on the second client amazon instance._**

-------------------------------------------------- 

# Load balancing using Nginx


You can use Nginx as a load balancer to balance the load. Nginx provided the incoming requests and sends it to the backend servers. To configure Nginx as a load balancer, you have two add two block of code to the nginx configration file.

### **Note: _Do this on the Master server_**

#### Step 1: Create loadbalancer.conf file on the Master server under /etc/nginx/conf.d/ .

```
sudo touch /etc/nginx/conf.d/loadbalancer.conf
```

#### Step2: Add the upstream group under the HTTP section under the configration file (/etc/nginx/conf.d/loadbalancer.conf). The upstream group is the group of server which comes under the load balancer. You can give any name to the group. Here I am going to give the name as "backend" and you have to set the vhost configration to receive trafficc from a particular domain name and route it to the upstream severs.

```
upstream backend {
    server $client_server_IP:80;
    server $client_server_IP:80;
}

server {
    listen 80;
    server_name domain.com www.domain.com;
    location / {
        proxy_pass http://backend;
    }
}

```
### Then validate the configration file using nginx -t and restart nginx


## **_Please note : Replace the domain.com with your domain name._**

-------------------------------------------------- 
# Enabling Let's encrypt on Amazon Linux with Nginx

Learn how to setup an Amazon Linux 2 EC2 instance with nginx to accept HTTPS requests. [Click here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/SSL-on-amazon-linux-2.html#letsencrypt)


## Certbot

Certbot is a free, open-source software tool for automatically using Let's Enrypt certificates on manaully-admininstrated website to enable HTTPS

#### Step1: Install Certbot on the EC2 instance.
```
sudo yum install -y certbot 
sudo yum install -y python-certbot-nginx
```

#### Step2: Please add an A record to your domain.com DNS. Replace the domain.com with your domain 

#### Step3: Setting up Certbot to obtain your Certificate. Run the following certboot command to to obtain your Certificate, replacing domain.com with your domain:

```
sudo certbot --nginx -d domian.com -d www.domian.com
```
#### **You may view the message below. When you run the above command, it deploys the certificate to VirtualHost automatically. You can use the [manual setup](#Manual-setup) configuration if the automated Deploying Certificate to VirtualHost fails.**

```
Deploying Certificate to VirtualHost /etc/nginx/conf.d/loadbalancer.conf
Deploying Certificate to VirtualHost /etc/nginx/conf.d/loadbalancer.conf
Redirecting all traffic on port 80 to ssl in /etc/nginx/conf.d/loadbalancer.conf
Redirecting all traffic on port 80 to ssl in /etc/nginx/conf.d/loadbalancer.conf

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations! You have successfully enabled https://domain.com and
https://www.domain.com
```

## Manual setup

> If you want to install this Let's encrypt on your current Loadbalancer.  If yes, follow the procedures above on your Master server and add the following code to the loadbalancer configuration file (/etc/nginx/conf.d/loadbalancer.conf).

```
server {
   listen 443 ssl;
   server_name domain_name;
   ssl_certificate /etc/letsencrypt/live/domain_name/cert.pem;
   ssl_certificate_key /etc/letsencrypt/live/domain_name/privkey.pem;
   ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

   location / {
      proxy_pass http://backend;
   }
}

```
#### **Note: Replace the paths ssl_certificate and ssl_certificate_key with your domain's SSL certificate. Replace your domain name as well.**

#### With the HTTPS-enabled you also have the option to enforce encryption to all connections to your load balancer. Simply update your server segment listening to port 80 with a server name and a redirection to your HTTPS port. Then remove or comment out the location portion as it’s no longer needed. See the example below.

```
server {
   listen 80;
   server_name domain_name;
   return 301 https://$server_name$request_uri;

   #location / {
   #   proxy_pass http://backend;
   #}
}
```
## Conclusion

In this article, I have explained how to install and configure Ngix on ec2 amzon-linux. Please contact me when you encounter any difficulty error while installing nginx loadbalancer on amzon-linux ec2-instance. Thank you!

### ⚙️ Connect with Me
<p align="center">
<a href="https://www.instagram.com/iamvyjith/"><img src="https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white"/></a>
<a href="https://www.linkedin.com/in/vyjith-ks-3bb8b7173/"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white"/></a>
