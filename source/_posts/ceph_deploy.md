---
title: ceph 部署与概念 
---

-存储类型：
1.NAS - Network Attached Storage 
2.SAN - storage area network
3.DAS - Direct Attached Storage

-存储协议：
1.NFS --linux 
2.SMB --windows

-存储方案：
1.PV:persistant volume (node)
2.PVC: persistant volume claim (pod)

理论：best 官网文档。 kuberneters 英文官网文档。
https://wenku.baidu.com/view/3f2d05c34b35eefdc9d333e7.html

kubernetes持久化存储：
https://baijiahao.baidu.com/s?id=1575913413903593&wfr=spider&for=pc

ceph/daemon 部署：
https://hub.docker.com/r/ceph/daemon/
https://www.jianshu.com/p/f08ed7287416
https://www.linuxidc.com/Linux/2017-10/148041.htm
https://www.kubernetes.org.cn/ingress


------实践步骤-------
体系结构
配置文件
部署环境配置
部署：
1. 部署 CEPH 的用户
ssh user@ceph-server
sudo useradd -d /home/{username} -m {username}
sudo passwd {username}

授权
echo "{username} ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/{username}
sudo chmod 0440 /etc/sudoers.d/{username}

2. 无密码 SSH 登录
ssh-keygen
ssh-copy-id {username}@node1
ssh-copy-id {username}@node2
ssh-copy-id {username}@node3

3. deploy mon
docker run -it --net=host \
--hostname=cephmonhost \
--name=cephtimon \
-v /etc/ceph:/etc/ceph \
-v /var/lib/ceph/:/var/lib/ceph/ \
-e MON_IP=172.20.36.231 \
-e CEPH_PUBLIC_NETWORK=172.20.36.0/24 \
ceph/daemon mon

ctrl+P+Q

4. deploy mgr
docker run -d --net=host \
--hostname=cephmgrhost \
--name=cephtimmgr \
-v /etc/ceph:/etc/ceph \
-v /var/lib/ceph/:/var/lib/ceph/ \
ceph/daemon mgr


5. distribut keyring
# ssh root@node2 mkdir -p /var/lib/ceph
# scp -r /etc/ceph root@node2:/etc
# scp -r /var/lib/ceph/bootstrap* root@node2:/var/lib/ceph


6. zap disk
docker run -d --privileged=true \
-v /dev/:/dev/ \
-e OSD_DEVICE=/dev/sdb \
ceph/daemon zap_device


7. deploy osd
docker run -it --net=host \
--hostname=cephosdhost \
--name=cephtimosd \
--pid=host \
--privileged=true \
-v /etc/ceph:/etc/ceph \
-v /var/lib/ceph/:/var/lib/ceph/ \
-v /dev/:/dev/ \
-e OSD_DEVICE=/dev/sdb \
ceph/daemon osd

8. deploy mds

docker run -d --net=host \
-v /var/lib/ceph/:/var/lib/ceph/ \
-v /etc/ceph:/etc/ceph \
-e CEPHFS_CREATE=1 \
ceph/daemon mds

9. deploy rgw

docker run -d --net=host \
-v /var/lib/ceph/:/var/lib/ceph/ \
-v /etc/ceph:/etc/ceph \
ceph/daemon rgw

