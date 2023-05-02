# Samba
-----------------

1. 패키지 설치

- dnf install samba samba-common samba-client

--------------------

2. 디렉터리 권한 설정하기

- mkdir -p /data/nas
- chmod -R /data/nas
- chown -R user:user /data/nas
- chcon -t samba_share_t /data/nas

------------------------

3. samba 설정 파일
[global]
workgroup = WORKGROUP
server string = Samba Server %v
netbios name = rocky-8
security = user
map to guest = bad user
dns proxy = no
ntlm auth = true

[Public]
path = /data/nas
browsable = yes
writable = yes
guest ok = yes
read only = no

----------------------

4. 설정파일 테스트

- testparm

5. 서비스 재시작

- systemctl start smb nmb
- systemctl enabled smb nmb
- systemctl status smb nmb

6. samba 계정 추가

- useradd smbuser
- smbpasswd -a smbuser

7. samba 그룹 추가 
- groupadd smb_group
- usermod -g smb_group smbuser

8. 디렉터리 권한 설정

mkdir -p /home/keonmu/share
chmod -R 770 /home/keonmu/share
chcon -t samba_share_t /home/keonmu/share
chown -R root:smb_group /home/keonmu/share

9. samba 권한 설정
[Private]
path = /home/keonmu/share
valid users = @smb_group
guset ok =no
writable = no 
browsable = yes

-----------------------------------------------

# SMB CLIENT

sudo dnf install samba-client

smbclient ‘\2.168.43.121\private’ -U smbuser
