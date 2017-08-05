# Udp2raw-tunnel
An Encrpyted,Anti-Replay,Multiplexed Udp Tunnel,tunnels udp traffic through raw socket,send/recv udp packet as raw packet with fake tcp/icmp header. Which can help you bypass udp blocking or udp qos. It also supports sending raw packet as udp packet,in this way you can just may use of the encrpyting and anti-replay feature.Nat supported in all 3 modes.

In tcp mode simulated 3-way hand-shake,simluated seq ack_seq implemented. Those tcp options are also implemented:MSS,sackOk,TS,TS_ack,wscale  

## Getting Started

### Prerequisites
linux host,root access

### Installing
download binary release from https://github.com/wangyu-/udp2raw-tunnel/releases

### Running 
assume your udp is blocked or being QOS-ed or just poorly supported.

assume your server ip is 44.55.66.77, you have a service listening on udp port 7777.
```
run at client side:
./udp2raw_amd64 -c -l0.0.0.0:3333  -r44.55.66.77:4096 -a -k "passwd" --raw-mode faketcp

run at server side:

./udp2raw -s -l0.0.0.0:4096 -r 127.0.0.1:7777  -a -k "passwd" --raw-mode faketcp

```
Now,your client and server established a tunnel thorough tcp port 4096. Connecting to udp port 3333 at client side  is equivalent with connecting to port 7777 at server side. No udp traffic will be exposed to outside.

## Advanced Topic

### Usage
```
udp2raw-tunnel
version: Aug  5 2017 21:03:54
repository: https://github.com/wangyu-/udp2raw-tunnel

usage:
    run as client : ./this_program -c -l local_listen_ip:local_port -r server_ip:server_port  [options]
    run as server : ./this_program -s -l server_listen_ip:server_port -r remote_ip:remote_port  [options]

common options,these options must be same on both side:
    --raw-mode            <string>        avaliable values:faketcp(default),udp,icmp
    -k,--key              <string>        password to gen symetric key,default:"secret key"
    --auth-mode           <string>        avaliable values:aes128cbc(default),xor,none
    --cipher-mode         <string>        avaliable values:md5(default),crc32,simple,none
    -a,--auto-rule                        auto add (and delete) iptables rule
    -g,--gen-rule                         generate iptables rule then exit
    --disable-anti-replay                 disable anti-replay,not suggested
client options:
    --source-ip           <ip>            force source-ip for raw socket
    --source-port         <port>          force source-port for raw socket,tcp/udp only
                                          this option disables port changing while re-connecting
other options:
    --log-level           <number>        0:never    1:fatal   2:error   3:warn 
                                          4:info (default)     5:debug   6:trace
    --log-position                        enable file name,function name,line number in log
    --disable-color                       disable log color
    --disable-bpf                         disable the kernel space filter,most time its not necessary
                                          unless you suspect there is a bug
    --sock-buf            <number>        buf size for socket,>=10 and <=10240,unit:kbyte,default:1024
    --seqmode             <number>        seq increase mode for faketcp:
                                          0:dont increase
                                          1:increase every packet
                                          2:increase randomly, about every 3 packets (default)
    -h,--help                             print this help message
```
### iptables rule
this programs sends packet via raw socket.In faketcp mode,Linux Kernel TCP packet processing has to be blocked by a iptables rule on both side,otherwise Kernel will automatically send RST for unrecongized TCP packet and you will sustain from stability/peformance problem.You can use -a option to let the program automatically add/del iptables rules on start/exit.You can also use the -g option to generate iptables rule and add it manually.

### cipher-mode and auth-mode 
Its suggested to use aes128cbc + md5 to obtain maxmized security.If you want to run the program on a router,you can try xor+simple,it can fool Packet Inspection by firewalls most time, but it cant protect you from serious attackers. Mode none is only for debug,its not suggest to set cipher-mode or auth-mode to none.

### seq-mode
the faketcp mode doest not behave 100% like a real tcp connection.ISP may be able to distinguish the simulated tcp traffic from real tcp traffic(though its costly). seq-mode can help you changed the seq increase behavior a bit. If you experienced problems,try to change the value. 
