# NFS 설정
--------------------
### NFS-Server

#### 1. NFS 설치
- yum install nfs-utils
--------------------
#### 2. 공유한 디스크 생성 및 공유폴더 지정
- Disk 20G 생성
- lvm 으로 생성 후 mkfs.ext4 /dev/sdb1번으로 파일시스템 지정
- mount /dev/sdb1 /usr1 로 마운트 진행 
- df -hT 로 마운트 상태 확인
--------------------
#### 3. vi /etc/exportfs 파일 지정
- /usr1 192.168.194.*(rw,sync) <br/>
  -> /usr1번에 대한 디렉터리를  192.168.194의 모든대역에 읽기,쓰기권한으로 열어준 걸 의미함 <br/> sync는 즉시 동기화 옵션을 말함 <br/>
--------------------       
- 설정옵션 <br/>
  rw : 읽기,쓰기 <br/>
  ro : 읽기전용 <br/>
  sync : 파일시스템 변경시 즉시 동기화 <br/>
  secure : 클라이언트의 마운트요청시 1024이하의 포트를 사용 <br/>
  noaccess : 액세스거부 <br/>
  root_squach : 클라이언트의 서버root권한 획득을 막기(default) 서버에서 생성된 파일을 수정못함 <br/>
  no_root_squash : 클라이언트 root계정과 서버의 root계정을 동일하게봄 서버에서 생성된 파일 수정가능 <br/>
  no_alll_squach : root를 제외하고 서버와 클라이언트의 사용자를 동일한 권한으로 설정 <br/>
  no_all_squach : root를 제외하고 서버와 클라이언트의 사용자들을 하나의 권한을 가지도록 설정 <br/>
--------------------  
#### 4. Directory에 권한 부여(일반 사용자 읽고 쓰기 가능)
- chmod 707 /usr1
--------------------
#### 5. exportfs의 수정 내용 반영
- exportfs -r
--------------------
#### 6. 서비스 기동
- systemctl start nfs-server
- systemctl enable nfs-server
--------------------
#### 7. 서비스 확인
- showmount -e 
- exportfs -v 
---------------------

### NFS-client
-------------------
Ubuntu에서 실습 진행

1. nfs 설치
- apt install nfs-common
-------------------
2. 마운트 지점 생성
- mkdir /usr1
-------------------
3 마운트 실행
- sudo mount 192.168.194.130:/usr1 /usr1
-------------------
4. fstab 등록
- 192.168.194.130:/usr1 /usr1 nfs sync 0 0
