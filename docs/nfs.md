## NFS Server setup
```
sudo yum install nfs-utils
sudo systemctl status nfs-server
sudo systemctl enable nfs-server
sudo systemctl start nfs-server
sudo systemctl status nfs-server
sudo mkdir -p /svc/nfs/kubedata
sudo chmod -R 777 /svc/nfs
sudo vi /etc/exports
```
[adminuser@justvm-haproxynode-0 ~]$ cat /etc/exports
```
/svc/nfs/kubedata       *(rw,sync,no_subtree_check,insecure)
```
```
sudo exportfs -rav
sudo vi /etc/exports
sudo exportfs -rav
sudo exportfs -v
showmount -e
ls -la /svc/nfs/kubedata
```

## Mount NFS Drive
```
yum install showmount
showmount -e 10.1.96.5
mount -t nfs 10.1.96.5:/svc/nfs/kubedata /mnt
mount | grep kubedata
cd /mnt
touch abc
ls -la
```
