# rpi
Setting up my Pi

##  Adding 2 disks

# create partitions
```
sudo parted /dev/nvme0n1 --script mklabel gpt mkpart primary ext4 0% 100%
sudo parted /dev/nvme1n1 --script mklabel gpt mkpart secondary ext4 0% 100%
```
# mount adding below lines to /etc/fstab
```
/dev/nvme0n1p1       /mnt/disk1      ext4     defaults,noatime  0       2
/dev/nvme1n1p1       /mnt/disk2      ext4     defaults,noatime  0       2

mount /mnt/disk1
mount /mnt/disk2

df -h
```
# move docker and containerd to use the new disk1
```
sudo systemctl stop docker
sudo systemctl stop containerd


sudo mkdir -p /mnt/disk1/docker /mnt/disk1/docker/overlay2 /mnt/disk1/docker/volumes /mnt/disk1/docker/containerd /mnt/disk1/docker/containerd/state
sudo chown -R root:root /mnt/disk1/docker
sudo chmod -R 711 /mnt/disk1/docker
```
# changes in docker deamon config
```
cat /etc/docker/daemon.json
{
  "data-root": "/mnt/disk1/docker"
}
```
# create a containerd config if it does not exist
```
sudo containerd config default | sudo tee /etc/containerd/config.toml
```
# add changes to containerd config
/etc/containerd/config.toml
```
2:root = '/mnt/disk1/docker/containerd'
138:    root = "/mnt/disk1/docker/containerd"
139:    state = "/mnt/disk1/docker/containerd/state"
```
# start them now
```
sudo systemctl daemon-reload
sudo systemctl start containerd
sudo systemctl start docker
sudo systemctl enable docker
```
## Test

```
df -k
docker pull nginx
df -k
```
Check if /mnt/disk1 has used more space. 

Have fun !