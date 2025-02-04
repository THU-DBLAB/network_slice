
<p align="center">
    <img style="border-style:1px;border-style:double;border-color:#8C8C8C" src="https://img-blog.csdnimg.cn/img_convert/1725e103892524230c3b09c9b060476c.png" width="500"/>
</p>

<p align="center">
    <img style="border-style:1px;border-style:double;border-color:#8C8C8C" src="https://imgur.com/dEX1ZFG.jpg" />
</p>

* Nova：透過配置及管理使用任何 Hypervisor 的虛擬機器，提供所需運算資源。
* Swift：提供了彈性的、高可用的分散式對象存儲服務，適合存儲大規模非結構化數據。
* Glance：提供可開機磁碟映像的登錄，以及儲存與擷取這些映像的服務。
* Horizon：提供簡易 Web 介面和管理，控制台為系統管理員和使用者提供圖形化介面，以供存取、配置及自動化雲端資源。可擴充設計更易與協力廠商產品和服務搭配使用。
* Cinder：提供Block資料存取，將持續區塊存取裝置對應至 OpenStack 運算執行個體，提供區塊儲存設備服務功能，並支援多種儲存解決方案。
* Keystone：提供身分驗證機制，提供集中式使用者目錄，並將其對應至使用者可存取的 OpenStack 服務。本模組可做為跨雲端作業系統的通用驗證機制，並可與現有的後端目錄服務進行整合。
* Neutron：提供網路管理功能，提供插入式、可擴充、由 API 驅動的系統，用以管理網路和 IP 位址。插件式架構可讓使用者充分善用其基本市售工具，或是支援廠商的進階網路服務。
---

- ## Environment requirement
*    **ubuntu** &nbsp;&nbsp;**`18.04` or lastest**
*    **devstack** **`lastest`** [:link:](https://docs.openstack.org/devstack/latest/)

---

```bash
# update and upgrade
$ sudo apt update && sudo apt upgrade -y
$ sudo apt install -y vim net-tools git
```
```bash
# add user name=stack
$ sudo useradd -s /bin/bash -d /opt/stack -m stack
$ echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
$ sudo su - stack
```

```bash
$ git clone https://opendev.org/openstack/devstack
$ cd devstack
# copy the local.conf file
$ cp samples/local.conf .

# find the device's IPv4
$ ifconfig 

$ vim local.conf
```

<p align="center">
    <img style="border-style:1px;border-style:double;border-color:#8C8C8C" src="https://imgur.com/YswSWc0.png" width="500"/>
</p>

- #### local.conf
```text
[[local|localrc]]
ADMIN_PASSWORD=secret
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
HOST_IP=device_IP
```

```bash
# edit inc/python
$ vim inc/python
```

- #### inc/python

```bash
# at line 198 add --ignore-installed
# Turn
$cmd_pip $upgrade \
# into 
$cmd_pip $upgrade --ignore-installed \
```
<p align="center">
    <img style="border-style:1px;border-style:double;border-color:#8C8C8C" src="https://imgur.com/wIwYQ3G.png" width="500"/>
</p>

- ### **Run devstack**
```bash
# install openstack
$ sudo chown -R stack:stack /opt/stack
$ ./stack.sh
```

**If there is Permission Error, Run ```$ sudo chown -R stack:stack /opt/stack``` and ```$ ./stack.sh``` again.** 

---
- ### **Install Finish**

<p align="center">
    <img style="border-style:1px;border-style:double;border-color:#8C8C8C" src="https://imgur.com/5eQusBh.png" width="500"/>
</p>

---

<p align="center">
    <img style="border-style:1px;border-style:double;border-color:#8C8C8C" src="https://imgur.com/E3W8Ilu.png" width="800"/>
</p>

---
- ## Add Router and interface for subnet

**Project -> Network -> Network Topology -> Create Router** 

<p align="center">
    <img style="border-style:1px;border-style:double;border-color:#8C8C8C" src="https://imgur.com/ERdP7mx.png" width="500"/>
</p>

---

- ## Add the ICMP, TCP ,UDP Ingress/Engress Security Group

**Project -> Network -> Security Groups -> Manage Rules** 

<p align="center">
    <img style="border-style:1px;border-style:double;border-color:#8C8C8C" src="https://imgur.com/WqgThqb.png" width="800"/>
</p>

---

- ## Add Key Pairs

**Project -> Compute -> Key pairs -> Create Key Pair** 

<p align="center">
    <img style="border-style:1px;border-style:double;border-color:#8C8C8C" src="https://imgur.com/gTicVf3.png" width="800"/>
</p>

Save the ```name.pem``` file

---
- ## Upload images

**Project -> Compute -> Images -> Create Image** 

```bash
# Download ubuntu bionic server image
$ wget https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img
```

<p align="center">
    <img style="border-style:1px;border-style:double;border-color:#8C8C8C" src="https://imgur.com/dzEUiJw.png" width="800"/>
</p>

---

- ## Launch Instance

**Project -> Compute -> Instances -> Launch Instance** 

1.  Details
    * Instance Name = Name
    * Count = 1
2.  Source
    * Allocated -> ubuntu
3.  Flavor
    * Allocated -> select one
4.  Network
    * Allocated -> shared-subnet
5.  Security Groups
    * Allocated -> default
6.  Key Pair
    * Allocated -> name

**Launch Instance and Waitting setup until ...**

<p align="center">
    <img style="border-style:1px;border-style:double;border-color:#8C8C8C" src="https://imgur.com/qgMIhDA.png" width="800"/>
</p>

---

- ## Associate Floating IP

**Project -> Compute -> Instances -> Launch Instance -> Associate Floating IP** 

<p align="center">
    <img style="border-style:1px;border-style:double;border-color:#8C8C8C" src="https://imgur.com/W8cExb2.png" width="800"/>
</p>

---

- ## SSH into Created Instance

```bash
$ ssh -i /path/my-key-pair.pem ubuntu@IP-address
# set DNS
$ sudo vim /etc/resolv.conf
# set nameserver 8.8.8.8
```

<p align="center">
    <img style="border-style:1px;border-style:double;border-color:#8C8C8C" src="https://imgur.com/SXqiRR1.png" width="600"/>
</p>


```bash
$ sudo apt update
$ sudo apt install iperf3 -y
```

<p align="center">
    <img style="border-style:1px;border-style:double;border-color:#8C8C8C" src="https://imgur.com/XVs5V8t.png" width="600"/>
</p>

---

```bash
# or run a web server
$ sudo apt install apache2
$ sudo mv /var/www/html/index.html /var/www/html/index.html.backup
$ sudo vim /var/www/html/index.html
```
- #### index.html
```htmlembedded
<!DOCTYPE html>
<html>
<head>
    <title>Web Server on Ubuntu</title>
</head>
    <body>
        <h1>My web server in ubuntu</h1>
        <p>My name is ____.</p>
    </body>
</html>
```

<p align="center">
    <img style="border-style:1px;border-style:double;border-color:#8C8C8C" src="https://imgur.com/vh7RXsp.png" width="800"/>
</p>

