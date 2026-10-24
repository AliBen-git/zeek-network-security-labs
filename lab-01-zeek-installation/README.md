# Lab 1 - Zeek Installation

# **What is Zeek?**

An open-source protocol analyzer and network security monitoring tool, Zeek was once known as Bro. It is intended to assist enterprises with real-time network traffic monitoring and analysis, offering information on network activity, potential security risks, and performance concerns. Due to its effectiveness in swiftly capturing and processing network data, Zeek is especially well-liked among cybersecurity experts and network managers.

# How to Install Zeek in Ubuntu?

Update and upgrade the ubuntu using apt.

```bash
sudo apt-get update
sudo apt-get upgrade
```

## Zeek official download page

Download the zeek source code from the official website (https://zeek.org/get-zeek/).



## Install dependencies using the below command.

```bash
sudo apt-get install cmake make gcc g++ flex bison libpcap-dev libssl-dev python3-dev swig zlib1g-dev

sudo apt install -y libzmq3-dev pkg-config
```

**Once all the dependencies are installed change the directory to the path where the Zeek source code file is downloaded and unzip the file.**

```bash
cd Downloads
tar -xzf zeek-version.tar.gz
```


## **Change the directory to the extracted file**

```bash
cd zeek-version
./configure
```



**Once the above command is done run the below commands. Note that this command takes time to execute.**

```bash
make
make install
```


## To use zeek as a service we need to add the zeek home directory to the bashrc file.

```bash
nano ~/.bashrc
```

**Add the line below or the home directory file zeek at the end of the file.**

```bash
export PATH=/usr/local/zeek/bin:$PATH
```

Save and exit the file and to apply changes made run source command and check zeek version and directory.

```bash
source ~/.bashrc
which zeek
zeek --version
```





If you are unable to see Zeek Version follow the steps below:

```bash
ls -l /usr/local/zeek/bin
```

If the above commands says cannot find then run the following command

```bash
sudo find / -type f -name zeek 2>/dev/null | head
```

Another error to look for

!image.png

```bash
unable to open database file: /usr/local/zeek/spool/state.db
```

When you are getting the above error run the following command, replace the username with your username to fix the permission issue

```bash
sudo chown -R aayush:aayush /usr/local/zeek
```

</aside>

**Now change the directory to /usr/local/zeek/etc check the what files are there in the directory.**

```bash
cd /usr/local/zeek/etc
ls
```


**Open new terminal window and check the ip using the below command check the network interface of the machine.**

```bash
ip a
```


We can see there are 2 interfaces, one is for loopback and other is for broadcast and more which is **enp0s1**. Note that this may vary from user to user. Note the interface name. Now in the previous window edit **node.cfg** file using nano and replace the interface name as shown below.

```bash
nano node.cfg
```


Once the file is saved check if the script is correct, using the below command.



```bash
sudo setcap cap_net_raw,cap_net_admin=eip /usr/local/zeek/bin/zeek
```

</aside>

```bash
zeekctl check
```

```bash
zeekctl check
```

Once you get “zeek scripts are ok.” at the end you can deploy zeek, using below command.

```bash
zeekctl deploy
```

Once zeek is started we can check the status using.

```bash
zeekctl status
```


**Now to view logs we can change the directory to /usr/local/zeek/logs/current**

```bash
cd /usr/local/zeek/logs/current
```

When we use the list command we can see the logs been generated.


**We can use tail command to view the logs,**

```bash
tail -f conn.log
```
