## 1、安装keepalived、haproxy

>所有master主机上安装

```shell
yum install keepalived haproxy -y
```

## 2、配置keepalived

```shell
# 其中一台角色设置为MASTER，其他设置为BACKUP 
STATE=MASTER
# 监听的网卡接口
INTERFACE=eth0
# 集群ID
ROUTER_ID=51
# 集群优先级，MASTER设置为101,BACKUP设置为100
PRIORITY=101
# 配置认证信息
AUTH_PASS=112233
# 设置VIP的地址
APISERVER_VIP=172.16.4.240/24
# 设置VIP的端口
APISERVER_DEST_PORT=8443

mkdir -p /etc/keepalived/

# 配置keepalived
cat >/etc/keepalived/keepalived.conf << EOF
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
}
vrrp_script check_apiserver {
  script "/etc/keepalived/check_apiserver.sh"
  interval 3
  weight -2
  fall 10
  rise 2
}

vrrp_instance VI_1 {
    state MASTER
    interface ${INTERFACE}
    virtual_router_id ${ROUTER_ID}
    priority ${PRIORITY}
    authentication {
        auth_type PASS
        auth_pass ${AUTH_PASS}
    }
    virtual_ipaddress {
        ${APISERVER_VIP} dev ${INTERFACE}
    }
    track_script {
        check_apiserver
    }
}
EOF

# 配置检测脚本
cat >/etc/keepalived/check_apiserver.sh<<EOF
#!/bin/sh

errorExit() {
    echo "*** $*" 1>&2
    exit 1
}

curl --silent --max-time 2 --insecure https://localhost:${APISERVER_DEST_PORT}/ -o /dev/null || errorExit "Error GET https://localhost:${APISERVER_DEST_PORT}/"
if ip addr | grep -q "${APISERVER_VIP}"; then
    curl --silent --max-time 2 --insecure https://${APISERVER_VIP}:${APISERVER_DEST_PORT}/ -o /dev/null || errorExit "Error GET https://${APISERVER_VIP}:${APISERVER_DEST_PORT}/"
fi
EOF
```

## 3、配置haproxy

```shell
# 设置VIP的端口
APISERVER_DEST_PORT=8443
# 设置源站的端口
APISERVER_SRC_PORT=6443
# 设置主机ID
HOST1_ID=centos-vm-4-241
HOST2_ID=centos-vm-4-242
HOST3_ID=centos-vm-4-243
# 设置主机IP
HOST1_ADDRESS=172.16.4.241
HOST2_ADDRESS=172.16.4.242
HOST3_ADDRESS=172.16.4.243

mkdir -p /etc/haproxy/
# 配置haproxy
cat >/etc/haproxy/haproxy.cfg<<EOF
# /etc/haproxy/haproxy.cfg
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    log /dev/log local0
    log /dev/log local1 notice
    daemon

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 1
    timeout http-request    10s
    timeout queue           20s
    timeout connect         5s
    timeout client          20s
    timeout server          20s
    timeout http-keep-alive 10s
    timeout check           10s

#---------------------------------------------------------------------
# apiserver frontend which proxys to the masters
#---------------------------------------------------------------------
frontend apiserver
    bind *:${APISERVER_DEST_PORT}
    mode tcp
    option tcplog
    default_backend apiserver

#---------------------------------------------------------------------
# round robin balancing for apiserver
#---------------------------------------------------------------------
backend apiserver
    option httpchk GET /healthz
    http-check expect status 200
    mode tcp
    option ssl-hello-chk
    balance     roundrobin
        server ${HOST1_ID} ${HOST1_ADDRESS}:${APISERVER_SRC_PORT} check
        server ${HOST2_ID} ${HOST2_ADDRESS}:${APISERVER_SRC_PORT} check
        server ${HOST3_ID} ${HOST3_ADDRESS}:${APISERVER_SRC_PORT} check
EOF
```

## 4、启动服务

```shell
systemctl enable haproxy --now
systemctl enable keepalived --now
```
