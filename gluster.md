# Gluster 구축 단계
------------------
## 사전준비
#### 구축 서버 4대
- Rocky Gluster Server 2대
- Rokcy Gluster Client 1대
- Ubuntu Gluster Client 1대
#### /etc/hosts 설정
- 192.168.194.130 keonmu2
- 192.168.194.131 keonmu1
#### hostname 수정
- 192.168.194.131 -> keonmu1 로 변경
- 192.168.194.130 -> keonmu2 로 변경
- hostnamectl set-hostname keonmu1 / keonmu2
#### Disk 할당
- Rocky Gluster Server 2대에 20GB disk 하나씩 할당
------------------
## Gluster Server 구축

#### 1. 디스크 마운트

- lvm 생성 
  할당한 Disk 20GB LVM 생성

Server 1 </br>
Disk /dev/sdb: 20 GiB, 21474836480 bytes, 41943040 sectors </br>
Units: sectors of 1 * 512 = 512 bytes </br>
Sector size (logical/physical): 512 bytes / 512 bytes </br>
I/O size (minimum/optimal): 512 bytes / 512 bytes </br>
Disklabel type: dos </br>
Disk identifier: 0x544084b8 </br>

Device     Boot Start      End  Sectors Size Id Type </br>
/dev/sdb1        2048 41943039 41940992  20G 8e Linux LVM </br>

Server 2 </br>
Disk /dev/sdc: 20 GiB, 21474836480 bytes, 41943040 sectors </br>
Units: sectors of 1 * 512 = 512 bytes </br>
Sector size (logical/physical): 512 bytes / 512 bytes </br>
I/O size (minimum/optimal): 512 bytes / 512 bytes </br>
Disklabel type: dos </br>  
Disk identifier: 0xf6b70051 </br>

Device     Boot Start      End  Sectors Size Id Type </br>
/dev/sdc1        2048 41943039 41940992  20G 8e Linux LVM </br>

- 파일시스템 포맷 </br>
  mkfs.ext4 /dev/sdb1
  mkfs.ext4 /dev/sdc1
  
- 마운트 진행 </br>
  vi /etc/fstab </br>
  /dev/sdb1 /data/vol1 ext4 defaults 0 0 </br>
  /dev/sdc1 /data/vol2 ext4 defaults 0 0 </br> 
  
  mount -a  </br> 
  /dev/sdb1           ext4       20G  1.2M   19G   1% /data/vol1 </br> 
  /dev/sdc1           ext4       20G  1.2M   19G   1% /data/vol2 </br> 
------------------
#### 2. 디렉터리 생성
- mkdir -p /data/vol1/brick0
- mkdir -p /data/vol2/brick0
------------------
#### 3. 패키지 설치
- dnf install centos-release-gluster9 </br>
------------------
#### 4. Repository 수정
- cd /etc/yum.repos.d/
  vi CentOS-Gluster-9.repo
  baseurl 수정 후 mirrorlist 주석처리    
[centos-gluster9] </br> 
name=CentOS-$releasever - Gluster 9 </br> 
#mirrorlist=http://mirrorlist.centos.org?arch=$basearch&release=$releasever&repo=storage-gluster-9 </br> 
baseurl=https://dl.rockylinux.org/vault/centos/8.5.2111/storage/x86_64/gluster-9/ </br> 
gpgcheck=1 </br> 
enabled=1 </br> 
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-Storage </br> 

- repository 확인
  [root@keonmu1 yum.repos.d]# dnf repolist </br> 
  레포지터리 ID                      레포지터리 이름 </br> 
  appstream                          Rocky Linux 8 - AppStream </br> 
  baseos                             Rocky Linux 8 - BaseOS </br> 
  centos-gluster9                    CentOS-8 - Gluster 9 </br> 
  extras                             Rocky Linux 8 - Extras </br> 
------------------
#### 5. Gluster Package 설치
- dnf install glusterfs glusterfs-libs glusterfs-server
- systemctl start glusterfsd.service
- systemctl enable glusterfsd.service
- systemctl status glusterfsd.service
- systemctl start glusterd.service
------------------
#### 6. 방화벽 설정
- systemctl status glusterfsd.service
- firewall-cmd --reload
- firewall-cmd --list-services
------------------
#### 7. Gluster 초기화

- gluster peer probe keonmu2 </br> 
  peer probe: success </br> 

- gluster peer status </br> 
  Number of Peers: 1 </br> 
  Hostname: keonmu2 </br> 
  Uuid: dc62999d-5554-4dc1-840d-a895ee3e6772 </br> 
  State: Peer in Cluster (Connected) </br> 

- gluster peer status (Server 2) </br> 
  Number of Peers: 1 </br> 
  Hostname: keonmu1 </br> 
  Uuid: 1722201d-a49b-44d9-b679-5767eab1de85 </br> 
  State: Peer in Cluster (Connected) </br> 
------------------
#### 8. Gluster 볼륨 생성
- gluster volume create myvolume replica 2 keonmu1:/data/vol1/brick0 keonmu2:/data/vol2/brick0 </br> 
  Replica 2 volumes are prone to split-brain. Use Arbiter or Replica 3 to avoid this. See:             
  http://docs.gluster.org/en/latest/Administrator%20Guide/Split%20brain%20and%20ways%20to%20deal%20with%20it/. 
  Do you still want to continue? </br> 
  (y/n) y </br> 
  volume create: myvolume: success: please start the volume to access data </br> 
------------------  
#### 9. Gluster 서비스 시작
- gluster volume start myvolume </br>
  volume start: myvolume: success </br>
------------------    
#### 10. Gluster 서비스 확인  
  - gluster volume status myvolume </br>
    Status of volume: myvolume </br>
    Gluster process                             TCP Port  RDMA Port  Online  Pid </br>
    ------------------------------------------------------------------------------ </br>
    Brick keonmu1:/data/vol1/brick0             49152     0          Y       35218 </br>
    Brick keonmu2:/data/vol2/brick0             49152     0          Y       35308 </br>
    Self-heal Daemon on localhost               N/A       N/A        Y       35235 </br>
    Self-heal Daemon on keonmu2                 N/A       N/A        Y       35325 </br>

    Task Status of Volume myvolume </br>
    ------------------------------------------------------------------------------ </br>
    There are no active volume tasks </br>

   - sudo gluster volume info </br>
     Volume Name: myvolume </br>
     Type: Replicate </br>
     Volume ID: 8677716f-4408-4a63-aac2-8b0b5a606076 </br>
     Status: Started </br>
     Snapshot Count: 0 </br>
     Number of Bricks: 1 x 2 = 2 </br>
     Transport-type: tcp </br>
     Bricks: </br>
     Brick1: keonmu1:/data/vol1/brick0 </br>
     Brick2: keonmu2:/data/vol2/brick0 </br>
     Options Reconfigured: </br>
     cluster.granular-entry-heal: on </br>
     storage.fips-mode-rchecksum: on </br>
     transport.address-family: inet </br>
     nfs.disable: on </br>
     performance.client-io-threads: off </br>
------------------  
## Gluster Client 구축

####1. 패키지 설치
-  dnf install glusterfs-client
------------------  
####2. Directory 생성
- mkdir /data
------------------  
####3. 마운트 진행
- mount.glusterfs keonmu1:/myvolume /data
------------------  
####4. 서비스 확인 </br>
[root@keonmu3 ~]# df -hT </br>
Filesystem          Type            Size  Used Avail Use% Mounted on </br>
devtmpfs            devtmpfs        359M     0  359M   0% /dev </br>
tmpfs               tmpfs           389M     0  389M   0% /dev/shm </br>
tmpfs               tmpfs           389M   12M  378M   3% /run </br>
tmpfs               tmpfs           389M     0  389M   0% /sys/fs/cgroup </br>
/dev/mapper/rl-root xfs              47G  5.2G   42G  12% / </br>
/dev/sda1           xfs            1014M  260M  755M  26% /boot </br>
tmpfs               tmpfs            78M   56K   78M   1% /run/user/1000 </br>
keonmu1:/myvolume   fuse.glusterfs   20G  201M   19G   2% /data </br>

파일 생성 테스트 </br>
서버 1/2에서 /data/vol1/brick0  , /data/vol2/brick0 확인하면 동일한 파일 생성 확인 가능 </br>
[root@keonmu3 data]# pwd </br>
 /data </br> </br>
[root@keonmu3 data]# ls -al </br>
합계 4  </br>
drwxr-xr-x.  4 root root 4096  5월  3 21:43 . </br>
dr-xr-xr-x. 18 root root  236  5월  3 14:06 .. </br>
-rw-r--r--.  1 root root    0  5월  3 21:43 keonmu1 </br>
