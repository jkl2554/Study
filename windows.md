# windows
symbolic link 만들기
```ps
## powershell
New-Item -ItemType SymbolicLink -Path "Link" -Target "Target"
## cmd
mklink /d "Link" "Target"
```