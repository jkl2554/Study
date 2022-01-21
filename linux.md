# linux
## echo에서 root파일쓰기
```s
echo "my text" > /path/to/text  ## root 권한 사용 불가
echo "my text" | sudo tee /path/to/text > /dev/null  ## 파일 쓰기 시 root로 씀
```

## systemd

```s
systemctl --user ## 유저별 서비스 설정

# /etc/systemd/system/ # root systemd 디렉토리
# /usr/lib/systemd/user/ # user systemd 디렉토리
# ~/.config/systemd/user # user custom systemd 디렉토리
```
## ubuntu iptables

```s
## IP tables를 이용해 80 443 포트 8080 8443에 각각 매핑
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
sudo iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 8443
sudo iptables -t nat -L PREROUTING

# Chain PREROUTING (policy ACCEPT)
# target     prot opt source               destination
# CNI-HOSTPORT-DNAT  all  --  anywhere             anywhere             ADDRTYPE match dst-type LOCAL
# REDIRECT   tcp  --  anywhere             anywhere             tcp dpt:http redir ports 8080
# REDIRECT   tcp  --  anywhere             anywhere             tcp dpt:https redir ports 8443
```
*iptables 변경 시 바로 적용되니 설정 시 주의 필요.  
```s
## IP tables를 이용해 80 443 포트 8080 8443에 매핑한 정보 제거
sudo iptables -t nat -D PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
sudo iptables -t nat -D PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 8443
sudo iptables -t nat -L PREROUTING

# Chain PREROUTING (policy ACCEPT)
# target     prot opt source               destination
# CNI-HOSTPORT-DNAT  all  --  anywhere             anywhere             ADDRTYPE match dst-type LOCAL
```
재 시작 시 자동 적용되게 저장  
```s
sudo iptables-save
```
