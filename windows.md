# windows
symbolic link 만들기
```ps
## powershell
New-Item -ItemType SymbolicLink -Path "Link" -Target "Target"
## cmd
mklink /d "Link" "Target"
```

DNS 관련 정보
```ps
nslookup # DNS 서버에 질의

ipconfig /displaydns # 현재 OS에 플러시된 DNS정보

ipconfig /flushdns # ipconfig /displaydns 에 나온정보 다시 업로드

ping 으로 테스트할때 ipconfig /displaydns에 나온 정보로 IP검색함
```