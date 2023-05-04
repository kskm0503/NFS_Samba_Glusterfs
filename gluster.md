# Gluster 구축 단계
------------------
## 사전준비

#### 구축 서버 4대
- Rocky Gluster Server 2대
- Rokcy Gluster Client 1대
- Ubuntu Gluster Client 1대
------------------
#### /etc/hosts 설정
- 192.168.194.130 keonmu2
- 192.168.194.131 keonmu1
------------------
#### hostname 수정
- 192.168.194.131 -> keonmu1 로 변경
- 192.168.194.130 -> keonmu2 로 변경
- hostnamectl set-hostname keonmu1 / keonmu2
------------------
### Disk 할당
- Rocky Gluster Server 2대에 20GB disk 하나씩 할당
------------------

## Gluster Server 구축

1.
