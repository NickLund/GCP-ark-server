# GCP-ark-server
How to make a dedicated ark server in GCP
### Instance Config
```
e2-standard-2
100GB; SSD persistent disk; debian
network tags: [unique firewall rule name, i.e. ark-server-fwr]
```
### VPC Firewall Settings:
```
Name:                 ark-rules
Targets:              Specified target tags
Target tags:          [unique firewall rule name listed above, i.e. ark-server-fwr]
Source filter:        IPv4 ranges
Source IPv4 ranges:   0.0.0.0/0
Protocols and ports:  UDP: 27015,7777,7778 (Optional - TCP: 27020 -> RCON for remote console server access)
```
### VM commands and instructions
```
$ sudo apt-get update
$ sudo apt-get install lib32gcc-s1
$ sudo useradd -m [new server user]
$ sudo passwd [new server user]
```
> [enter password]
```
$ su - [new server user]

$ mkdir ~/Steam && cd ~/Steam
$ curl -sqL "https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz" | tar zxvf -
$ ./steamcmd.sh
```
**NOTE: I installed the ark server within the Steam directory. You probably shouldn't do this.**
```
Steam> force_install_dir ./ark/
Steam> login anonymous
Steam> app_update 376030 validate
Steam> exit

$ mkdir ~/Steam/ark/Engine/Binaries/ThirdParty/SteamCMD/Linux && cd ~/Steam/ark/Engine/Binaries/ThirdParty/SteamCMD/Linux
$ curl -sqL "https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz" | tar zxvf -
$ ./steamcmd.sh

Steam> exit

$ exit

$ cd /home/[new server user]/Steam/ark/Engine/Binaries/ThirdParty/SteamCMD/Linux
$ sudo ln -s /home/[new server user]/Steam/steamapps/ steamapps
$ sudo chown -h [new server user]:[new server user] steamapps
```
**NOTE: I added this step, but not sure if it did anything**
```
$ sudo nano /etc/sysctl.conf
```
> fs.file-max=100000
```
$ sudo sysctl -p /etc/sysctl.conf
$ sudo nano /etc/security/limits.conf
```
> \*		soft	nofile		1000000 <br>
> \*		hard	nofile		1000000
```
$ sudo nano /etc/pam.d/common-session
```
> session required	pam_limits.so
<br> **END NOTE STEP**
```
$ sudo nano /etc/systemd/system/ark-dedicated.service
```
> [Unit]
> Description=ARK: Survival Evolved dedicated server
> Wants=network-online.target
> After=syslog.target network.target nss-lookup.target network-online.target
> 
> [Service]
> ExecStartPre=/home/[new server user]/Steam/steamcmd +force_install_dir ./ark +login anonymous +app_update 376030 +quit
> ExecStart=/home/[new server user]/Steam/ark/ShooterGame/Binaries/Linux/ShooterGameServer TheIsland?listen -automanagedmods -server -log
> WorkingDirectory=/home/[new server user]/Steam/ark/ShooterGame/Binaries/Linux
> LimitNOFILE=100000
> ExecReload=/bin/kill -s HUP $MAINPID
> ExecStop=/bin/kill -s INT $MAINPID
> User=[new server user]
> Group=[new server user]
> 
> [Install]
> WantedBy=multi-user.target
```
$ sudo systemctl enable ark-dedicated
$ sudo systemctl start ark-dedicated
