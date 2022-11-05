# linux

## 환경변수
설정 
```s
export <환경변수 명>=값

```
영구 적용  
```s
## /etc/bash.bashrc root
## /home/사용자명/.bashrc 특정사용자
## 후행에 추가
USER_BASHRC_PATH=~/.bashrc
echo "export <환경변수 명>=값" >> $USER_BASHRC_PATH
## sudo 권한 필요 시 
ROOT_BASHRC_PATH=/etc/bash.bashrc
echo "export <환경변수 명>=값" | sudo tee -a $ROOT_BASHRC_PATH >/dev/null
```

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

EOF 정리  
```s
cat <<EOF | <cammand> -file - 
파일 내용
EOF

cat <<EOF > <filename>
파일 내용
EOF
## 예
cat <<EOF | kubectl apply -f - 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: $DNS_LABEL-ingress
  namespace: keycloak
  annotations:
    kubernetes.io/ingress.class: $INGRESS_CLASS
    cert-manager.io/cluster-issuer: letsencrypt-staging
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  tls:
  - hosts:
    - $DNS_LABEL.koreacentral.cloudapp.azure.com
    secretName: tls-secret
  rules:
  - host: $DNS_LABEL.koreacentral.cloudapp.azure.com
    http:
      paths:
      - backend:
          service:
            name: keycloak-http
            port: 
              number: 80
        path: /(.*)
        pathType: Prefix
EOF
```
Shell script 실행 파일 위치
```s
## 실행 파일의 디렉토리
dirname $(readlink -f $0)
```

linux user
```s
sudo -i #user 계정으로 로그인 root 환경변수 
sudo su - #user 계정으로 로그인 root 환경변수 
sudo su #user 계정으로 로그인 user 환경변수
su root #root 계정으로 로그인 user 환경변수
su - #root계정 로그인 root 환경변수
```