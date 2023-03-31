# Databox Gateway Install
## 선행 조건
- Powershell Az Module 설치: https://learn.microsoft.com/powershell/azure/install-az-ps

## Azure vm deploy
```ps
$serviceName = "DataBoxGW"
$rgname = "DataBoxGW-RG";
$loc = "koreacentral";
New-AzResourceGroup -Name $rgname -Location $loc -Force;

# VM Profile & Hardware
$vmname = "WinCoreVM";
$vnetname = "myVnet";
$vnetAddress = "10.0.0.0/16";
$subnetname = "slb" + $serviceName;
$subnetAddress = "10.0.1.0/24";
$SourcAddressPrefix = "$(Invoke-RestMethod -Uri "ipconfig.me" -Headers @{Accept = "application/json"})"
$NSGName = $subnetname + "-NSG";
$OSDiskName = $vmname + "-osdisk";
$OSDiskSKU="Standard_LRS"
$pipName = $vmname + "-pip"
$NICName = $vmname + "-nic";
$OSDiskSizeinGB = 256;
$DataDiskName = $vmname + "-datadisk";
$DataDiskSKU="Standard_LRS"
$DataDiskSizeinGB = 2048;
$VMSize = "Standard_D4s_v3";
$PublisherName = "MicrosoftWindowsServer";
$Offer = "WindowsServer";
$SKU = "2022-datacenter-core-g2";

# Credential
$user = "hostuser";
$password = "qwer1234!@#$";
$securePassword = $password | ConvertTo-SecureString -AsPlainText -Force;  
$cred = New-Object System.Management.Automation.PSCredential ($user, $securePassword);

# Network resources
$nsgRuleHQ = New-AzNetworkSecurityRuleConfig -Name "HQRule" -Protocol "Tcp"  -Direction "Inbound" -Priority 1001 -SourceAddressPrefix $SourcAddressPrefix -SourcePortRange "*" -DestinationAddressPrefix "*" -DestinationPortRange "*" -Access Allow;
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $RGName -Location $loc -Name $NSGName  -SecurityRules $nsgRuleHQ;
$frontendSubnet = New-AzVirtualNetworkSubnetConfig -Name $subnetname -AddressPrefix $subnetAddress  -NetworkSecurityGroup $nsg;
$vnet = New-AzVirtualNetwork -Name $vnetname -ResourceGroupName $rgname -Location $loc -AddressPrefix $vnetAddress -Subnet $frontendSubnet;
$pip = New-AzPublicIpAddress -Name $pipName -ResourceGroupName $rgname -Location $loc -AllocationMethod Dynamic;
$nic = New-AzNetworkInterface -Name $NICName -ResourceGroupName $RGName -Location $loc -SubnetId $vnet.Subnets[0].Id -EnableAcceleratedNetworking -PublicIpAddressId $PIP.Id;

# VM creation
$vmConfig = New-AzVMConfig -VMName $vmName -VMSize $VMSize;
Set-AzVMOperatingSystem -VM $vmConfig -Windows -ComputerName $vmName -Credential $cred;
Set-AzVMSourceImage -VM $vmConfig -PublisherName $PublisherName -Offer $Offer -Skus $SKU -Version "latest";
Add-AzVMNetworkInterface -VM $vmConfig -Id $nic.Id;
Set-AzVMOSDisk -VM $vmConfig -Windows -DiskSizeInGB $OSDiskSizeinGB -Name $OSDiskName -StorageAccountType $OSDiskSKU -CreateOption "FromImage";
Add-AzVMDataDisk -VM $vmConfig -DiskSizeInGB $DataDiskSizeinGB -Name $DataDiskName -StorageAccountType $DataDiskSKU -LUN 0 -CreateOption "Empty";
Set-AzVMBootDiagnostic -VM $vmConfig -Enable;
New-AzVM -ResourceGroupName $RGName -Location $loc -VM $vmConfig;
```

## in Host VM
### install hyper-v
CoreOS인 경우 설치
```ps
# After Windows Server core version 1903, Can use Hyper-v Manger
Add-WindowsCapability -Online -Name ServerCore.AppCompatibility~~~~0.0.1.0
```
Hyper-v 설치
```ps
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
```
Hyper-v Manager 실행
```ps
virtmgmt.msc
```
Disk partition설정
```ps
diskmgmt.msc
## Extend Volume Os disk to maximum
## Create Volume Data disk to Disk letter F: and maximum volume size
```
### Port 선점 서비스 종료
```ps
Stop-Service winRM -force
Set-Service winRM -StartupType disabled
Stop-Service LanmanServer -force
Set-Service -Name LanmanServer -StartupType disabled
Restart-Computer -Force
```
### Network 설정
```ps
$SWNAME="natSW"
$NATNAME="LocalNAT"
$NATCIDR="192.168.100.0/24"

$NATGWIP="$(($NATCIDR.split("/")[0]).substring(0,$($NATCIDR.split("/")[0]).Length-1))1"
$NATLANGTH="$($NATCIDR.split("/")[1])"

$LOCALVMIP="$(($NATCIDR.split("/")[0]).substring(0,$($NATCIDR.split("/")[0]).Length-1))2"

New-VMSwitch -SwitchName $SWNAME -SwitchType "Internal"

$IFINDEX=$(Get-NetAdapter -Name "vEthernet ($SWNAME)").ifIndex

New-NetIPAddress -IPAddress $NATGWIP -PrefixLength $NATLANGTH -InterfaceIndex "$IFINDEX"
New-NetNat -Name "$SWNAME-NAT" -InternalIPInterfaceAddressPrefix "$NATCIDR"
Add-NetNatStaticMapping -ExternalIPAddress "0.0.0.0/24" -ExternalPort 443 -Protocol TCP -InternalIPAddress "$LOCALVMIP" -InternalPort 443 -NatName "$SWNAME-NAT"
Add-NetNatStaticMapping -ExternalIPAddress "0.0.0.0/24" -ExternalPort 5985 -Protocol TCP -InternalIPAddress "$LOCALVMIP" -InternalPort 5985 -NatName "$SWNAME-NAT"
Add-NetNatStaticMapping -ExternalIPAddress "0.0.0.0/24" -ExternalPort 5986 -Protocol TCP -InternalIPAddress "$LOCALVMIP" -InternalPort 5986 -NatName "$SWNAME-NAT"
Add-NetNatStaticMapping -ExternalIPAddress "0.0.0.0/24" -ExternalPort 445 -Protocol TCP -InternalIPAddress "$LOCALVMIP" -InternalPort 445 -NatName "$SWNAME-NAT"
Add-NetNatStaticMapping -ExternalIPAddress "0.0.0.0/24" -ExternalPort 2049 -Protocol TCP -InternalIPAddress "$LOCALVMIP" -InternalPort 2049 -NatName "$SWNAME-NAT"
```

### Data Box Gateway VHDX Download
```ps
$url = "https://aka.ms/dbe-vhdx-2012" 
$FileName = (Split-Path -Path $url -Leaf)
$TEMP = "D:\"
$SaveTO= "C:\"

Start-Job -Name "Download $FileName" -ScriptBlock {param($url, $FileName, $TEMP, $SaveTO)
    Try{
        (New-Object System.Net.WebClient).DownloadFile($url, "$TEMP\$FileName.zip")
        Expand-Archive -Path "$TEMP\$FileName.zip" -DestinationPath "$SaveTO\$FileName"
    }
    Catch{
        Write-Host "Couldn't download $FileName.zip"
        $_
    }
} -ArgumentList $url, $FileName, $TEMP, $SaveTO
```
### Data Box Gateway가상머신 만들기
- cpu: 4 core
- memory: 8196 MB
- Network Adapter: `$SWNAME`
#### 디스크 추가작업
- Disk Type: Dynamically Expending
- Location: F:\
- Size: 2048 GB
## in DBGW Guest VM
```ps
Get-HcsIpAddress
# OperationalStatus : Up
# Name : Ethernet
#
# Set-HcsIpAddress –Name "Ethernet" –IpAddress $LOCALVMIP –Netmask 255.255.255.0 –Gateway $NATGWIP
Set-HcsIpAddress –Name "Ethernet" –IpAddress 192.168.100.2 –Netmask 255.255.255.0 –Gateway 192.168.100.1
```
## in My PC
### Data Box Gateway 홈페이지에서 설정
- `https://$pip`로 접속
- 네트워크 설정에서 퍼블릭 DNS 등록(8.8.8.8, 8.8.4.4)
### Azure Portal Data Box Gateway 리소스 생성
- Resource Group: `$rgname`
- Name: mydataboxgw
- Location: Southeast Asia
### Data Box Gateway Add Share
- Name: myshare
- Type: SMB
- Storage Account: `<Storage account name>`
- Storage Service: Block blob
- Select blob container: Create new, myshare
- All privilege local user: Create new
- User name: blobuser
- Password: qwer1234!@#$


##### 참고: https://learn.microsoft.com/ko-kr/azure/databox-gateway/data-box-gateway-overview