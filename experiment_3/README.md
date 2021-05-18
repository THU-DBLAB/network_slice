<p align="center">

# Experiment 3 - QoS

</p>

<p align="center">
    <img style="border-style:1px;border-style:double;border-color:#8C8C8C" src="https://github.com/xxionhong/network_slice/blob/main/experiment_2/img/2021-01-09%20task2.png?raw=true" width="500"/>
</p>

### `In Terminal`

```bash
# start the ryu-manager
$ ryu-manager --verbose ryu/ryu/app/simple_switch_13.py
```
```bash
# start the mininet
$ sudo mn --custom network_slice/experiment_3/exp3_topo.py --topo mytopo --mac --switch ovs,protocols=OpenFlow13 --controller remote
```
### `In Mininet`
```bash
# open Xterm for h1 h2 h3 h4 in mininet 
$ xterm h1 h2 h3 h4
```
### `In Xterm h4`
```bash
# in h4
$ iperf -s
```
### `In Xterm h1 h2 h3`

```bash
# in h1 h2 h3
$ iperf -c 10.0.0.4
```

 :pushpin: **To see the transmission bandwidth from h1, h2, h3 to h4 before we set the Qos parameter.**

### `In Terminal`

```bash
# set Qos for Port s1-eth4 with 3 queues (q0 q1 q2),Qos Type is linux-htb
$ sudo ovs-vsctl -- set Port s1-eth4 qos=@newqos -- \
--id=@newqos create QoS type=linux-htb queues:0=@q0 queues:1=@q1 queues:2=@q2 -- \
--id=@q0 create Queue other-config:max-rate=9000000 -- \
--id=@q1 create Queue other-config:max-rate=6000000 -- \
--id=@q2 create Queue other-config:max-rate=3000000
# 9000000 = 9 Mb, 6000000 = 6 Mb, 3000000 = 3 Mb

# show the port s1-eth4 detail
$ sudo ovs-vsctl list port s1-eth4

# show the qos
$ sudo ovs-vsctl list qos {uuid}
# or
$ sudo ovs-appctl qos/show s1-eth4

# show the queue
$ sudo ovs-vsctl list queue {uuid}

# show the flowentry in s1
$ sudo ovs-ofctl -O openflow13 dump-flows s1

# modify the flowentry meet the match set action to different queue (q0 q1 q2)
$ sudo ovs-ofctl -O OpenFlow13 mod-flows s1 "table=0, priority=1, in_port="s1-eth1", dl_src=00:00:00:00:00:01, dl_dst=00:00:00:00:00:04, actions=set_queue:0,output:"s1-eth4""
$ sudo ovs-ofctl -O OpenFlow13 mod-flows s1 "table=0, priority=1, in_port="s1-eth2", dl_src=00:00:00:00:00:02, dl_dst=00:00:00:00:00:04, actions=set_queue:1,output:"s1-eth4""
$ sudo ovs-ofctl -O OpenFlow13 mod-flows s1 "table=0, priority=1, in_port="s1-eth3", dl_src=00:00:00:00:00:03, dl_dst=00:00:00:00:00:04, actions=set_queue:2,output:"s1-eth4""
```
### `In Xterm h1 h2 h3`

```bash
# in h1 h2 h3 to check the different
$ iperf -c 10.0.0.4
```

<p align="center">
    <img style="border-style:1px;border-style:double;border-color:#8C8C8C" src="https://github.com/xxionhong/network_slice/blob/main/experiment_3/img/2020-10-15%20215853.jpg?raw=true" width="700"/>
</p>
 
 :pushpin: **The transmission bandwidth is restrict in the queue max rate.**

### `Optional, In Terminal`

```bash
# kill all Qos and Queue
$ sudo ovs-vsctl -- --all destroy QoS -- --all destroy Queue
# kill single qos and queue
$ sudo ovs-vsctl remove qos {uuid} queue {queue_num}
$ sudo ovs-vsctl destroy queue {uuid}
```

---

# :bulb: Hint :

- Qos Table

|QoS type|Description|Configuration|
|---|----|--|
|linux-htb|Linux **hierarchy token bucket** classifier. See [tc-htb(8)](http://linux.die.net/man/8/tc-htb ) and the [HTB manual](http://luxik.cdi.cz/~devik/qos/htb/manual/userg.htm ) for information on how this classifier works and how to configure it.|max-rate|
|linux-hfsc|Linux **Hierarchical Fair Service Curve** classifier. [See](http://linux-ip.net/articles/hfsc.en/) for information on how this classifier works.|max-rate|
|linux-sfq|Linux **Stochastic Fairness Queueing** classifier. See [tc-sfq(8)](http://linux.die.net/man/8/tc-sfq) for information on how this classifier works.|perturb, quantum|
|linux-codel|Linux **Controlled Delay** classifier. See [tc-codel(8)](http://man7.org/linux/man-pages/man8/tc-codel.8.html) for information on how this classifier works.
|linux-fq_codel|Linux **Fair Queuing with Controlled Delay** classifier. See [tc-fq_codel(8)](http://man7.org/linux/man-pages/man8/tc-fq_codel.8.html) for information on how this classifier works.
|linux-netem|Linux **Network Emulator** classifier. See [tc-netem(8)](http://man7.org/linux/man-pages/man8/tc-netem.8.html) for information on how this classifier works.|latency, limit, loss|
|linux-noop|Linux **No operation.** By default, Open vSwitch manages QoS on all of its configured ports. This can be helpful, but sometimes administrators prefer to use other software to manage QoS. This type prevents Open vSwitch from changing the QoS configuration for a port.
|egress-policer|A DPDK egress policer algorithm using the **DPDK rte_meter library**. The rte_meter library provides an implementation which allows the metering and policing of traffic. |cir, cbs, eir, ebs|
|trtcm-policer|A DPDK egress policer algorithm using **RFC 4115’s Two-Rate, Three-Color marker**. It’s a two-level hierarchical policer which first does a color-blind marking of the traffic at the queue level, followed by a color-aware marking at the port level. At the end traffic marked as Green or Yellow is forwarded, Red is dropped. For details on how traffic is marked, see [RFC 4115](https://datatracker.ietf.org/doc/html/rfc4115). 
- **[ovs-vswitchd :link:](https://man7.org/linux/man-pages/man5/ovs-vswitchd.conf.db.5.html)**
- **[PICA8 Configuring QoS scheduler :link:](https://docs.pica8.com/display/PicOS211sp/Configuring+QoS+scheduler)**
- **[基於Open vSwitch的傳統限速和SDN限速:link:](https://www.sdnlab.com/23289.html)**
- **[Open vSwitch之QoS的實現 :link:](https://www.sdnlab.com/19208.html)**
- **[OVS QoS流量控制 (DPDK):link:](https://blog.csdn.net/sinat_20184565/article/details/93376574)**

---
:pushpin: **OVS does not Support !!! Only for PICA8**

```bash
# Round Robin (example)
$ sudo ovs-vsctl -- set Port s1-eth4 qos=@newqos -- \
--id=@newqos create QoS type=PRONTO_ROUND_ROBIN queues:0=@q0 queues:1=@q1 queues:2=@q2 -- \
--id=@q0 create Queue other-config:min-rate=100000000 other-config:max-rate=200000000 -- \
--id=@q1 create Queue other-config:min-rate=200000000 other-config:max-rate=400000000 -- \
--id=@q2 create Queue other-config:min-rate=400000000 other-config:max-rate=600000000
```

```bash
# Weighted Round Robin (example)
$ sudo ovs-vsctl -- set Port s1-eth4 qos=@newqos -- \
--id=@newqos create QoS type=PRONTO_WEIGHTED_ROUND_ROBIN queues:0=@q0 queues:1=@q1 queues:2=@q2 -- \
--id=@q0 create Queue other-config=max-rate=300000000,weight=5 -- \
--id=@q1 create Queue other-config=max-rate=200000000,weight=3 -- \
--id=@q2 create Queue other-config=max-rate=100000000,weight=1
```

```bash
# Weighted Fair Queuing (example)
$ sudo ovs-vsctl -- set Port s1-eth4 qos=@newqos -- \
--id=@newqos create QoS type=PRONTO_WEIGHTED_FAIR_QUEUING queues:0=@q0 queues:1=@q1 queues:2=@q2 -- \
--id=@q0 create Queue other-config=max-rate=300000000,weight=5 -- \
--id=@q1 create Queue other-config=max-rate=200000000,weight=3 -- \
--id=@q2 create Queue other-config=max-rate=100000000,weight=1
```


---

<p align="center">

# Homework

</p>

### 1. Topology

<p align="center">
    <img style="border-style:1px;border-style:double;border-color:#8C8C8C" src="https://github.com/THU-DBLAB/network_slice/blob/main/experiment_3/img/img-HW.jpg?raw=true" width="700"/>
</p>
透過 Mininet 自訂出如上圖的拓譜。

### 2. Add Flow

透過 ```sudo ovs-ofctl -O OpenFlow13 add-flow {Switch} {Contents}``` 來增加每個 Switch 中的 Flowentries，使得 **4 個 Host** 能夠互相連接。

### 3. Meter Table
設定 **任意 Switch** 中的 **Meter Table** 使得 **H1 (client)** 與 **H4 (server)** 之間的 bandwidth 限制為 **20 Mbps**。

### 4. QoS
設定 **S3** 的 **QoS**，使得 **H1 (server)** 對於 **H2 與 H3 (clients)** 的bandwidth 限制為 **10 Mbps** 與 **15 Mbps**。
