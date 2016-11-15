#openstack 网络

##修改计算节点
>[ovs]

>enable_tunneling = True

>tunnel_type = vxlan

>integration_bridge = br-int

>local_ip = 192.168.81.181

>bridge_mappings = external:br-ex

>把bridge_mappings = external:br-ex注释掉，然后重启ovs agent

>systemctl restart neutron-openvswitch-agent.service


##创建网络
###创建网络（外网）
1. 创建外网
>neutron net-create ext-net --shared --router:external True --provider:physical_network external --provider:network_type flat

2. 创建外网子网
 >neutron subnet-create ext-net --name ext-subnet --allocation-pool start=192.168.79.150,end=192.168.79.200 --disable-dhcp --gateway 192.168.79.2  192.168.79.0/24

 >start到end这个网段是给floting用的，也就是如果你的虚拟机如果想上外网，必须要有一个floating IP才可以上面

3. 创建内网
 >neutron net-create demo-net

4. 创建内网子网
>neutron subnet-create demo-net --name demo-subnet --gateway 192.168.1.1 192.168.1.0/24 
 
 >这个网段就是虚拟机的内部网，你可以通过网络节点登录到路由器茶看到
 
5. 创建路由
 >neutron router-create demo-router

6. 把内网的子网挂载到路由上
 >neutron router-interface-add demo-router demo-subnet

7. 给路由设置上网网关，其实就是加上外网
 >neutron router-gateway-set demo-router ext-net

如果是网络上面对于内网到路由是添加接口，对于外网到路由是设置网关

8. 创建虚拟机
 >nova boot --flavor m1.tiny --image cirros-0.3.1-x86_32 --nic net-id=$内网子网ID
 --security-group default  demo-instance1

9. 给虚拟机分配floating IP

10. 在网络节点登录到路由器 
 >查看：ip netns
 
 >进入：ip netns exec qrouter-57f10528-1a2e-40d9-9f0a-c0cf7fc2ebad bash
 
 >查看路由：ip route ls
 
 >可以看到之前内网的路由：192.168.1.0/24 dev qr-b8bc3cc3-7d  proto kernel  scope link  src 192.168.1.1.如果虚拟机是启动的，查看实例的虚拟ip，那么可以ping通，floating ip也是

##排错

1. 查看各个agen是否正常:查看alive字段，如果有问题，重启网络几点上面的neutron服务

>neutron agent-list

2. 创建虚拟机失败，提示no avalid host（没有可用的虚拟机）

>nova service-list

>nova host-list
