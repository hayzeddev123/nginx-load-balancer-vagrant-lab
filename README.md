# Nginx Load Balancer Lab with Multiple Vagrant Servers

**Author:** Adeniyi Abdulazeez (`hayzeddev123`)  
**Date:** August 2026

---

## 1. Introduction

This project demonstrates how to create a simple load-balanced web environment using Vagrant, VirtualBox, Ubuntu, and Nginx.

The lab consists of one Nginx load balancer and three backend web servers. The load balancer distributes incoming requests among the three servers using Nginx's default round-robin method.

---

## 2. Objectives

* Create four Ubuntu virtual machines using Vagrant.
* Configure three machines as web servers.
* Configure one machine as an Nginx load balancer.
* Configure private networking between the machines.
* Test connectivity and verify load balancing.

---

## 3. Technologies Used

* **VirtualBox** – Virtualization
* **Vagrant** – VM creation and management
* **Ubuntu 22.04 (Jammy)** – Operating system
* **Nginx** – Web server and load balancer
* **PowerShell** – Command-line management

---

## 4. Network Architecture

| VM    | Role            | IP Address     |
|-------|-----------------|----------------|
| nginx | Load Balancer   | 192.168.56.10  |
| web1  | Web Server 1    | 192.168.56.11  |
| web2  | Web Server 2    | 192.168.56.12  |
| web3  | Web Server 3    | 192.168.56.13  |

```
              Client
                 |
                 v
        Nginx Load Balancer
          192.168.56.10
                 |
       +---------+---------+
       |         |         |
       v         v         v
    Web 1     Web 2     Web 3
   .56.11     .56.12    .56.13
```

---

## 5. Vagrant Configuration

A `Vagrantfile` was created to define the four virtual machines and assign their private IP addresses.

Each machine used the `ubuntu/jammy64` box. Provisioning scripts were also configured to automatically install Nginx.

The web servers were configured to display different messages:

```
This is Web Server 1
This is Web Server 2
This is Web Server 3
```

This made it easy to identify which server handled each request.

---

## 6. Provisioning

Two provisioning scripts were used:

```
provision/
├── nginx.sh
└── webserver.sh
```

The scripts automatically installed Nginx and configured the web server pages.

The virtual machines were started using:

```bash
vagrant up
```

**Starting the Vagrant machines:**

![Vagrant Up](Screenshot%202026-08-16%20132418.png)

The final status showed all four machines running:

```
nginx    running
web1     running
web2     running
web3     running
```

---

## 7. Testing the Web Servers

Connectivity between the load balancer and the backend servers was tested using `curl`.

```bash
curl http://192.168.56.11
curl http://192.168.56.12
curl http://192.168.56.13
```

The servers returned:

```
This is Web Server 1
This is Web Server 2
This is Web Server 3
```

This confirmed that all three backend servers were accessible.

---

## 8. Nginx Load Balancer Configuration

The Nginx load balancer was configured using `/etc/nginx/sites-available/default`.

The important configuration was:

```nginx
upstream web_servers {
    server 192.168.56.11:80;
    server 192.168.56.12:80;
    server 192.168.56.13:80;
}

server {
    listen 80;
    listen [::]:80;

    server_name _;

    location / {
        proxy_pass http://web_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

The `upstream` block defines the three backend servers, while `proxy_pass` forwards incoming requests to them.

---

## 9. Configuration Verification

The Nginx configuration was tested using:

```bash
sudo nginx -t
```

During setup, some initial issues were encountered (service not found, missing default config):

![Nginx Setup Issues](Screenshot%202026-08-16%20132512.png)

After correcting the configuration and restarting:

![Nginx Active](Screenshot%202026-08-16%20132620.png)

The final result was:

```
syntax is ok
test is successful
Active: active (running)
```

---

## 10. Load Balancing Test

Multiple requests were sent to the load balancer:

```bash
curl http://192.168.56.10
```

The responses included different servers (round-robin):

```
This is Web Server 2
This is Web Server 3
This is Web Server 1
This is Web Server 2
This is Web Server 3
This is Web Server 1
```

This confirmed that Nginx was successfully distributing requests across all three backend servers.

---

## 11. Browser Testing

The load balancer was also tested from the host computer by opening:

```
http://192.168.56.10
```

Refreshing the page generated requests that were handled by different backend servers.

**Browser showing different backend servers:**

![Browser Test](Screenshot%202026-08-16%20133534.png)

---

## 12. Access Log Verification

The Nginx access logs were used to further verify traffic distribution.

```bash
sudo tail -f /var/log/nginx/access.log
```

**Access logs from the three web servers showing distributed requests:**

![Access Logs](Screenshot%202026-08-16%20134232.png)

Requests generated by refreshing the load balancer appeared in the backend server logs.

---

## 13. Problems Encountered

During the setup, several issues were encountered:

* **VirtualBox Host-Only network conflict** – The existing Host-Only adapter was inspected and reused.
* **Provisioning script path error** – The scripts were moved into the correct `provision/` directory.
* **SSH connection problems** – Vagrant SSH configuration and forwarded ports were checked.
* **No route to host** – The private network configuration was corrected so the servers could communicate.
* **Nginx permissions / missing config** – `sudo` was required and the default configuration was properly set up.

These issues were resolved successfully.

---

## 14. Results

The load balancer lab was successfully completed.

The final setup consisted of:

```
Client
   |
   v
Nginx Load Balancer
192.168.56.10
   |
   +---- Web Server 1 (192.168.56.11)
   |
   +---- Web Server 2 (192.168.56.12)
   |
   +---- Web Server 3 (192.168.56.13)
```

Testing confirmed that all three web servers received requests through the Nginx load balancer using round-robin distribution.

---

## 15. Conclusion

The project successfully demonstrated how Nginx load balancing works in a multi-server environment. Vagrant was used to create and provision the virtual machines, while Nginx handled both web serving and traffic distribution.

The successful responses from all three web servers confirmed that the load balancer was functioning correctly and distributing requests using the round-robin method.

**Lab Status: Successfully completed and verified.**
