# linux

## 환경변수
설정 
```sh
export <환경변수 명>=값

```
### 영구 적용  
```sh
## /etc/bash.bashrc root
## /home/사용자명/.bashrc 특정사용자
## 후행에 추가
USER_BASHRC_PATH=~/.bashrc
echo "export <환경변수 명>=값" >> $USER_BASHRC_PATH
## sudo 권한 필요 시 
ROOT_BASHRC_PATH=/etc/bash.bashrc
echo "export <환경변수 명>=값" | sudo tee -a $ROOT_BASHRC_PATH >/dev/null
```

### `.bashrc` vs `.bash_profile`
```sh
## .bashrc 는 비 로그인 Shell 에서 실행 .bash_profile 는 로그인 Shell 에서 실행
## 로그인 shell 비 로그인 shell 확인
################### 로그인 shell .bashrc, .bash_profile 모두 실행됨
prompt> echo $0
-bash # "-" is the first character. Therefore, this is a login shell.

prompt> shopt login_shell
login_shell     on

######################

prompt> bash # Enter NOT a login shell

################### 비 로그인 shell 진입 시 .bashrc만 실행됨
prompt> echo $0
bash # "-" is NOT the first character. This is NOT a login shell

prompt> shopt login_shell
login_shell     off

```
#### 참고문서 
* https://www.gnu.org/software/bash/manual/html_node/Bash-Startup-Files.html#Bash-Startup-Files

## echo에서 root파일쓰기
```sh
echo "my text" > /path/to/text  ## root 권한 사용 불가
echo "my text" | sudo tee /path/to/text > /dev/null  ## 파일 쓰기 시 root로 씀
```

## systemd

```sh
systemctl --user ## 유저별 서비스 설정

# /etc/systemd/system/ # root systemd 디렉토리
# /usr/lib/systemd/user/ # user systemd 디렉토리
# ~/.config/systemd/user # user custom systemd 디렉토리
```
## ubuntu iptables

```sh
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
```sh
## IP tables를 이용해 80 443 포트 8080 8443에 매핑한 정보 제거
sudo iptables -t nat -D PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
sudo iptables -t nat -D PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 8443
sudo iptables -t nat -L PREROUTING

# Chain PREROUTING (policy ACCEPT)
# target     prot opt source               destination
# CNI-HOSTPORT-DNAT  all  --  anywhere             anywhere             ADDRTYPE match dst-type LOCAL
```
재 시작 시 자동 적용되게 저장  
```sh
sudo iptables-save
```

EOF 정리  
```sh
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
```sh
## 실행 파일의 디렉토리
dirname $(readlink -f $0)
```

linux user  
```sh
sudo -i #user 계정으로 로그인 root 환경변수 
sudo su - #user 계정으로 로그인 root 환경변수 
sudo su #user 계정으로 로그인 user 환경변수
su root #root 계정으로 로그인 user 환경변수
su - #root계정 로그인 root 환경변수
```

## 파일 내용 바꾸기(sed)  
```sh
cat << EOF >myclient.yaml
mytest_\${VERSION}
EOF
## mytest_\${VERSION}
sed 's/${VERSION}/4.4.24-23/g' myclient.yaml
## mytest_4.4.24-23
```

## set 명령어  
```sh
set -o # 현재 설정된 옵션 보기
set -o <options> # 옵션 설정
set +o <options> # 옵션 해제
# ex
set -o vi # 커맨드라인 vi형태 명령으로 변경 default는 대부분 emacs
# 설정 변경 시 해당 shell 에서만 적용되니 .bashrc 등에서 미리 설정 해 두면 좋다.
```

n