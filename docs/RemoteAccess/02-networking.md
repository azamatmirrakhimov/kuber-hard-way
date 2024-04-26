~~~
{
  SERVER_IP=172.31.31.106
  NODE_1_IP=172.31.18.85
  NODE_1_SUBNET=10.200.0.0/24
  NODE_2_IP=172.31.24.235
  NODE_2_SUBNET=10.200.1.0/24
  NODE_3_IP=172.31.24.235
  NODE_3_SUBNET=10.200.2.0/24
  WORKER1_IP=172.31.24.130
  WORKER1_SUBNET=10.200.0.0/24
  WORKER2_IP=172.31.18.157
  WORKER2_SUBNET=10.200.1.0/24
  WORKER3_IP=172.31.22.46
  WORKER3_SUBNET=10.200.2.0/24
}
~~~

~~~
ssh root@server <<EOF
  ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
  ip route add ${NODE_2_SUBNET} via ${NODE_2_IP}
  ip route add ${NODE_3_SUBNET} via ${NODE_3_IP}
  ip route add ${WORKER1_SUBNET} via ${WORKER1_IP}
  ip route add ${WORKER2_SUBNET} via ${WORKER2_IP}
  ip route add ${WORKER3_SUBNET} via ${WORKER3_IP}
EOF
~~~

~~~
ssh root@node-1 <<EOF
  ip route add ${NODE_2_SUBNET} via ${NODE_2_IP}
  ip route add ${NODE_3_SUBNET} via ${NODE_3_IP}
  ip route add ${WORKER1_SUBNET} via ${WORKER1_IP}
  ip route add ${WORKER2_SUBNET} via ${WORKER2_IP}
  ip route add ${WORKER3_SUBNET} via ${WORKER3_IP}
EOF
~~~
