![IP地址规划表.png](https://s1.ax2x.com/2018/03/30/tC5Cn.png)


# 基本环境配置
1. 配置网络、主机名
修改和添加/etc/sysconfig/network-scripts/ifcfg-enp\*（具体的网口）文件。
   1. controller节点
```
配置网络：
enp8s0: 192.168.100.10
DEVICE=enp8s0
TYPE=Ethernet
ONBOOT=yes
NM_CONTROLLED=no
BOOTPROTO=static
IPADDR=192.168.100.10
PREFIX=24
GATEWAY=192.168.100.1

enp9s0: 192.168.200.10
DEVICE=enp9s0
TYPE=Ethernet
ONBOOT=yes
NM_CONTROLLED=no
BOOTPROTO=static
IPADDR=192.168.200.10
PREFIX=24

配置主机名：
# hostnamectl set-hostname controller
  按ctrl+d 退出  重新登陆
```
   2. controller节点
```
  配置网络：
  enp8s0: 192.168.100.20
  DEVICE=enp8s0
  TYPE=Ethernet
  ONBOOT=yes
  NM_CONTROLLED=no
  BOOTPROTO=static
  IPADDR=192.168.100.20
  PREFIX=24
  GATEWAY=192.168.100.1`

  enp9s0: 192.168.200.20
  DEVICE=enp9s0
  TYPE=Ethernet
  ONBOOT=yes
  NM_CONTROLLED=no
  BOOTPROTO=static
  IPADDR=192.168.200.20
  PREFIX=24

  配置主机名：
  # hostnamectl set-hostname compute
  按ctrl+d 退出  重新登陆
```
2. 配置yum源
```
#Controller和compute节点
   1. yum源备份
#mv /etc/yum.repos.d/\*  /opt/
   2. 创建repo文件
【controller】
在/etc/yum.repos.d创建centos.repo源文件
[centos]
name=centos
baseurl=file:///opt/centos
gpgcheck=0
enabled=1
[iaas]
name=iaas
baseurl=file:///opt/iaas-repo
gpgcheck=0
enabled=1

【compute】
在/etc/yum.repos.d创建centos.repo源文件
[centos]
name=centos
baseurl=ftp://192.168.100.10/centos
gpgcheck=0
enabled=1
[iaas]
name=iaas
baseurl=ftp://192.168.100.10/iaas-repo
gpgcheck=0
enabled=1

   3. 挂载iso文件
【挂载CentOS-7-x86_64-DVD-1511.iso】
[root@controller ~]# mount -o loop CentOS-7-x86_64-DVD-1511.iso  /mnt/
[root@controller ~]# mkdir /opt/centos
[root@controller ~]# cp -rvf /mnt/* /opt/centos/
[root@controller ~]# umount  /mnt/

【挂载XianDian-IaaS-v2.0-1228.iso】
[root@controller ~]# mount -o loop XianDian-IaaS-v2.0-1228.iso  /mnt/
[root@controller ~]# cp -rvf /mnt/* /opt/
[root@controller ~]# umount  /mnt/

   4. 搭建ftp服务器，开启并设置自启
[root@controller ~]# yum install vsftpd –y
[root@controller ~]# vi /etc/vsftpd/vsftpd.conf
添加anon_root=/opt/
保存退出

[root@controller ~]# systemctl start vsftpd
[root@controller ~]# systemctl enable vsftpd

   5. 关闭防火墙并设置开机不自启
【controller/compute】
systemctl stop firewalld
systemctl disable firewalld

   6. 清除缓存，验证yum源
【controller/compute】
# yum clean all
# yum list
```

3. 编辑环境变量
```
# controller和compute节点
# yum install iaas-xiandian -y
编辑文件/etc/xiandian/openrc.sh,此文件是安装过程中的各项参数，根据每项参数上一行的说明及服务器实际情况进行配置。
HOST_IP=192.168.100.10
HOST_NAME=controller
HOST_IP_NODE=192.168.100.20
HOST_NAME_NODE=compute
RABBIT_USER=openstack
RABBIT_PASS=000000
DB_PASS=000000
DOMAIN_NAME=demo（自定义）
ADMIN_PASS=000000
DEMO_PASS=000000
KEYSTONE_DBPASS=000000
GLANCE_DBPASS=000000
GLANCE_PASS=000000
NOVA_DBPASS=000000
NOVA_PASS=000000
NEUTRON_DBPASS=000000
NEUTRON_PASS=000000
METADATA_SECRET=000000
INTERFACE_NAME=enp9s0（外网网卡名）
CINDER_DBPASS=000000
CINDER_PASS=000000
TROVE_DBPASS=000000
TROVE_PASS=000000
BLOCK_DISK=md126p4（空白分区名）
SWIFT_PASS=000000
OBJECT_DISK=md126p5（空白分区名）
STORAGE_LOCAL_NET_IP=192.168.100.20
HEAT_DBPASS=000000
HEAT_PASS=000000
CEILOMETER_DBPASS=000000
CEILOMETER_PASS=000000
AODH_DBPASS=000000
AODH_PASS=000000
```
快速编辑命令
```
sed 's/^#//g' openrc.sh -i
sed 's/^#/##/g' openrc.sh -i
sed 's/PASS=/PASS=000000/g' openrc.sh  -i
```

4. 执行脚本安装
   - #controller&#compute *基础环境*
   `iaas-pre-host.sh`
   - #controller *Mysql*
   `iaas-install-mysql.sh`
   - #controller *Keystone*
   `iaas-install-keystone.sh`
   - #controller *Geystone*
   `iaas-install-Glance.sh`
   - #controller&#compute *Nova*
   `iaas-install-nova-controller.sh`
   `iaas-install-nova-conmpute.sh`
   - #controller&#compute *Neutron&Gre*
   `iaas-install-neutron-controller.sh`
   `iaas-install-neutron-conmpute.sh`
   `iaas-install-neutron-controller-gre.sh`
   `iaas-install-neutron-conmpute-gre.sh`
   - #controller *Dashboard*
   `iaas-install-dashboard.sh`

5. 访问Dashboard
  `http://controller/dashboard`
