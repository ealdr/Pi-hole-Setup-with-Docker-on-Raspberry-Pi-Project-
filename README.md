# Pi-hole Setup with Docker on Raspberry Pi Project

This README contains a full write up of the steps and processes I took to set up and run [Pi-hole](https://pi-hole.net/) on a Raspberry Pi 3 using [Docker](https://www.docker.com/).  
I created this as part of a personal project to gain hands on experience with containerization, self-hosting, and DNS level ad blocking.

---

## 1. What is Pi-hole

Pi-hole is a network-wide ad blocker, internet tracker blocker and has an large list of known malware infected websites blocked, which updates weekly.

---

## 2. What is Docker

Docker is a popular open-source platform that makes it easy to build, run, and manage applications in containers. A container is like a virtual machine, it packages an application along with everything it needs to run, but doesn't simulate the entire computer. You can run multiple applications on the same machine even if they require different software environments, without conflicts.

---

## Hardware and Software I used

- Raspberry Pi 3 b  
- Raspberry Pi OS (64-bit)  
- Docker + Docker Compose  
- Pi-hole (latest Docker image)

---

## Installation Process

I started by putting a fresh install of Pi OS onto the Pi via my PC and a Mico SD card reader. I then booted up the Raspberry PI and check for any updates by typing `sudo apt update` followed by `sudo apt full-upgrade` in the command prompt.

Instead of doing the regular install:  
(**Instructions:** https://github.com/pi-hole/pi-hole/#one-step-automated-install)
`curl -sSL https://install.pi-hole.net | bash`

---

I decided to install it with [Docker compose](https://docs.docker.com/compose/)
(**Instructions:** https://hub.docker.com/r/pihole/pihole). 
As if I ever wanted to remove Pi-hole in the future, it wont interfere with the other dependencies when uninstalling from the Pi.

To install Docker compose type `sudo apt-get install docker-compose` into the command prompt.

---

First I needed to install [Vim](https://roboticsbackend.com/install-use-vim-raspberry-pi/), which is a text editor. To do this, on my Raspberry Pi I entered into the command prompt `sudo apt-get install vim`. This installed Vim along with any required dependencies.


After it had installed, I typed `cd Desktop` which means I'm changing the directory I'm currently in to the desktop directory. 

Next, in the desktop directory I entered `vim docker-compose.yml` this creates (or opens if already exists) a text file in the Vim editor.

Now in the vim editor I pasted the Docker Image:

``` yaml # More info at https://github.com/pi-hole/docker-pi-hole/ and https://docs.pi-hole.net/
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      # DNS Ports
      - "53:53/tcp"
      - "53:53/udp"
      # Default HTTP Port
      - "80:80/tcp"
      # Default HTTPs Port. FTL will generate a self-signed certificate
      - "443:443/tcp"
      # Uncomment the below if using Pi-hole as your DHCP Server
      #- "67:67/udp"
    environment:
      # Set the appropriate timezone for your location (https://en.wikipedia.org/wiki/List_of_tz_database_time_zones), e.g:
      TZ: 'Europe/London'
      # Set a password to access the web interface. Not setting one will result in a random password being assigned
      FTLCONF_webserver_api_password: 'correct horse battery staple'
    # Volumes store your data between container upgrades
    volumes:
      # For persisting Pi-hole's databases and common configuration file
      - './etc-pihole:/etc/pihole'
      # Uncomment the below if you have custom dnsmasq config files that you want to persist. Not needed for most starting fresh with Pi-hole v6. If you're upgrading from v5 you and have used this directory before, you should keep it enabled for the first v6 container start to allow for a complete migration. It can be removed afterwards
      #- './etc-dnsmasq.d:/etc/dnsmasq.d'
    cap_add:
      # See https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
      # Required if you are using Pi-hole as your DHCP server, else not needed
      - NET_ADMIN
    restart: unless-stopped
```
Before saving, I changed the default password on line 19 `FTLCONF_webserver_api_password: 'correct horse battery staple'`

(I can also change the password at anytime by typing `docker exec pihole pihole -a -p <new password>`)

To save the yml, press `Esc`, type a colon (`:`), type `wq` and press `Enter`

I then ran the container by typing `docker-compose up`. To stop at anytime, I can type `docker-compose down`

Now Pi-hole was running, to access the admin pannel I needed to get my PI's Ip via `hostname -I` or `ip a`. With the IP address I went onto to my web browser and entered: `https://MY_IP/admin/`

On the login page I entered the password that I entered into the Docker Image earlier and logged into the dashboard.

On my Windows computer, I changed the DNS to point to my Pi-hole’s IP address by going to Settings > Network & Internet > Status > Change adapter options > Wi-Fi/Ethernet > Properties > IPv4. I selected "Use the following DNS server addresses" and typed the same IP address I use to access the Pi-hole dashboard into the Preferred DNS server field. Now, my computer sends and receives all its DNS queries through the Raspberry Pi running Pi-hole. Optionally you can set it for all your devices at once by setting your routers DNS to the Pi's IP address.

---

## Dashboard 
In the dashboard you can view how many addresses are blocked, the default is around 250,000, this included known malware links, and advert domains.

Other helpful sections in the dashboard:
- Total queries
- Queries blocked (number and %)
- Ability to add & take away from the black and allow list
- Timestamp and domain of requested in the query logs

---

## Tests I ran
To evaluate and observe the effectiveness of Pi-hole on my computer, I ran an [AdBlock tester](https://adblock-tester.com/) under three different setups:

### Router firewall settings only
Baseline test — relying only on the router's default ad/privacy protections. On here the Adblocker test tells me I score 38/100 points which is relatively bad.


<img width="183" height="32" alt="image" src="https://github.com/user-attachments/assets/7ccc8981-01a8-4488-9d34-1427dde1a61f" />



### Router + Pi-hole
Pi-hole and the router, without any browser based ad blockers. This scored 69/100 which is considered okay.

<img width="183" height="32" alt="image" src="https://github.com/user-attachments/assets/62235fdb-e7df-4a79-941f-74efb3499efe" />

### Router + Pi-hole + AdBlock Chrome Extension
Combined Pi-hole DNS filtering with a browser-based blocker. This scored 77/100 which is excellent.

<img width="183" height="32" alt="image" src="https://github.com/user-attachments/assets/9c4dd39f-8042-4e58-9416-7c022a6cf493" />

I also found adding the bitdefender anti-tracker chrome extension increases the total score to 92/100 which is more than necessary.

<img width="183" height="32" alt="image" src="https://github.com/user-attachments/assets/b1a859ad-62e6-440d-a31d-ed6ed19a2bd1" />





