# Storclilogs
To collect Storcli logs for Windows and ESX servers
Import-Module Posh-SSH
$hostname = Read-Host "Enter the server name"

if($hostname -like "*vsrv*")
{

Write-Host "`nEntering the Powershell session of" $hostname "......."

$session = New-PSSession -ComputerName $hostname
Enter-PSSession -Session $session

$server = $hostname -replace "\.us\.wal-mart\.com$", ""

mkdir "\\$hostname\C$\temp\$server" -Force | Out-Null

Copy-Item -Path \\chont10155us.homeoffice.wal-mart.com\C$\temp\storcli64.exe -Destination \\$hostname\C$\Temp



Write-Host "`nstorcli  file copied to the server"

Invoke-Command -Session $session -ScriptBlock {


cd \temp

$server1 = $using:server

Write-Host "`n Controllers information `n************************************************************************** `n"

./storcli64 show

$controller = Read-Host "Enter the controller (0 or 1 or both)"
if($controller -eq "0")
{

 Write-Host "`nCollecting Storcli Logs..........."
./storcli64.exe /c0 show all >> C:\temp\$server1\showall.txt
./storcli64.exe /c0 show events >> C:\temp\$server1\events0.txt
./storcli64.exe /c0 show termlog >> C:\temp\$server1\termlog.txt
./storcli64.exe /c0/bbu show all >> C:\temp\$server1\bbu_show.txt
./storcli64.exe /c0/cv show all >> C:\temp\$server1\cvshow.txt
./storcli64.exe /c0/dall show all >> C:\temp\$server1\dallshow.txt
./storcli64.exe /c0/eall show all >> C:\temp\$server1\eallshow.txt
./storcli64.exe /c0/eall/sall show all >> C:\temp\$server1\sallshow.txt
./storcli64.exe /c0/vall show all >> C:\temp\$server1\vallshow.txt

}

elseif($controller -eq "1")
{

 Write-Host "`nCollecting Storcli Logs..........."
./storcli64.exe /c1 show all >> C:\temp\$server1\showall.txt
./storcli64.exe /c1 show events >> C:\temp\$server1\events0.txt
./storcli64.exe /c1 show termlog >> C:\temp\$server1\termlog.txt
./storcli64.exe /c1/bbu show all >> C:\temp\$server1\bbu_show.txt
./storcli64.exe /c1/cv show all >> C:\temp\$server1\cvshow.txt
./storcli64.exe /c1/dall show all >> C:\temp\$server1\dallshow.txt
./storcli64.exe /c1/eall show all >> C:\temp\$server1\eallshow.txt
./storcli64.exe /c1/eall/sall show all >> C:\temp\$server1\sallshow.txt
./storcli64.exe /c1/vall show all >> C:\temp\$server1\vallshow.txt

}

elseif($controller -eq "both")
{

 Write-Host "`nCollecting Storcli Logs..........."
./storcli64.exe /call show all >> C:\temp\$server1\showall.txt
./storcli64.exe /call show events >> C:\temp\$server1\events0.txt
./storcli64.exe /call show termlog >> C:\temp\$server1\termlog.txt
./storcli64.exe /call/bbu show all >> C:\temp\$server1\bbu_show.txt
./storcli64.exe /call/cv show all >> C:\temp\$server1\cvshow.txt
./storcli64.exe /call/dall show all >> C:\temp\$server1\dallshow.txt
./storcli64.exe /call/eall show all >> C:\temp\$server1\eallshow.txt
./storcli64.exe /call/eall/sall show all >> C:\temp\$server1\sallshow.txt
./storcli64.exe /call/vall show all >> C:\temp\$server1\vallshow.txt

}
else
{
Write-Host "invalid input"
}

}

Write-Host "`nCopying data to chont10155us.homeoffice.wal-mart.com"

Copy-Item -Path "\\$hostname\C$\temp\$server" -Destination "\\chont10155us.homeoffice.wal-mart.com\C$\temp\Storcliscript" -Recurse -Force

Remove-Item -Path "\\$hostname\C$\temp\$server" -Recurse -Force

Write-host "`nStorcli logs collected and saved under C:\temp\Storcliscript\$server`n"

Exit-PSSession
Remove-PSSession $session

}

elseif($hostname -like "*vsh*")
{

Connect-VIServer -Server $hostname -User root -Password inBu1ld! | Out-Null
Get-VMHostService -VMHost $hostname | Where-Object { $_.Key -eq "TSM-SSH" } | Start-VMHostService | Out-Null
Write-Host "`nSSH Started`n"

$Username = "root"
$Password = "inBu1ld!"
$localFilePath = "C:\temp\Storcliscript\vmware-storcli.vib"      # Path to the file on your Windows machine
$remoteFilePath = "/tmp"

$cred = New-Object System.Management.Automation.PSCredential($username, (ConvertTo-SecureString $password -AsPlainText -Force))

Write-Host "`nCopying Storcli.vib and installing........"

Set-SCPItem -ComputerName $hostname -Credential $cred -Path $localFilePath -Destination $remoteFilePath -AcceptKey


$sshSession1 = New-SSHSession -ComputerName $hostname -Credential $cred -AcceptKey -ConnectionTimeout 1000

$commands = @(
    "esxcli software acceptance set --level=CommunitySupported",
    "esxcli software vib install -v /tmp/vmware-storcli.vib --no-sig-check --force"
    
)

foreach ($command in $commands) {
    $result = Invoke-SSHCommand -SessionId $sshSession1.SessionId -Command $command -TimeOut 1000
    Write-Host $result.Output 
}

Remove-SSHSession -SSHSession $sshSession1 | Out-Null


Write-Host "`n Controllers information `n************************************************************************** `n"



$Command1 = "/opt/lsi/storcli/storcli show"

$sshSession = New-SSHSession -ComputerName $hostname -Credential $cred -AcceptKey
$result = Invoke-SSHCommand -SessionId $sshSession.SessionId -Command $command1
Write-Output $result.Output


Remove-SSHSession -SSHSession $sshSession | Out-Null

$controller = Read-Host "Enter the controller (0 or 1 or both)"

if($controller -eq "0")
{
Write-Host "`nCollecting Storcli Logs..........."
$sshSession2 = New-SSHSession -ComputerName $hostname -Credential $cred -AcceptKey

$commands = @(
"/opt/lsi/storcli/storcli /c0 show all > /opt/lsi/storcli/showall.txt" 
"/opt/lsi/storcli/storcli /c0 show events > /opt/lsi/storcli/events.txt"
"/opt/lsi/storcli/storcli /c0 show termlog > /opt/lsi/storcli/termlog.txt"
"/opt/lsi/storcli/storcli /c0/bbu show all > /opt/lsi/storcli/bbu_show.txt"
"/opt/lsi/storcli/storcli /c0/cv show all > /opt/lsi/storcli/cvshow.txt"
"/opt/lsi/storcli/storcli /c0/dall show all > /opt/lsi/storcli/dallshow.txt"
"/opt/lsi/storcli/storcli /c0/eall show all > /opt/lsi/storcli/eallshow.txt"
"/opt/lsi/storcli/storcli /c0/eall/sall show all > /opt/lsi/storcli/sallshow.txt"
"/opt/lsi/storcli/storcli /c0/vall show all > /opt/lsi/storcli/vallshow.txt"
    
)

foreach ($command in $commands) {
    $result = Invoke-SSHCommand -SessionId $sshSession2.SessionId -Command $command
    #Write-Host $result.Output 
}
Remove-SSHSession -SSHSession $sshSession2 | Out-Null
}

elseif($controller -eq "1")
{
Write-Host "`nCollecting Storcli Logs..........."
$sshSession2 = New-SSHSession -ComputerName $hostname -Credential $cred -AcceptKey

$commands = @(
"/opt/lsi/storcli/storcli /c1 show all > /opt/lsi/storcli/showall.txt" 
"/opt/lsi/storcli/storcli /c1 show events > /opt/lsi/storcli/events.txt"
"/opt/lsi/storcli/storcli /c1 show termlog > /opt/lsi/storcli/termlog.txt"
"/opt/lsi/storcli/storcli /c1/bbu show all > /opt/lsi/storcli/bbu_show.txt"
"/opt/lsi/storcli/storcli /c1/cv show all > /opt/lsi/storcli/cvshow.txt"
"/opt/lsi/storcli/storcli /c1/dall show all > /opt/lsi/storcli/dallshow.txt"
"/opt/lsi/storcli/storcli /c1/eall show all > /opt/lsi/storcli/eallshow.txt"
"/opt/lsi/storcli/storcli /c1/eall/sall show all > /opt/lsi/storcli/sallshow.txt"
"/opt/lsi/storcli/storcli /c1/vall show all > /opt/lsi/storcli/vallshow.txt"
    
)

foreach ($command in $commands) {
    $result = Invoke-SSHCommand -SessionId $sshSession2.SessionId -Command $command
    #Write-Host $result.Output 
}
Remove-SSHSession -SSHSession $sshSession2 | Out-Null
}

elseif($controller -eq "both")
{
Write-Host "`nCollecting Storcli Logs..........."
$sshSession2 = New-SSHSession -ComputerName $hostname -Credential $cred -AcceptKey

$commands = @(
"/opt/lsi/storcli/storcli /call show all > /opt/lsi/storcli/showall.txt" 
"/opt/lsi/storcli/storcli /call show events > /opt/lsi/storcli/events.txt"
"/opt/lsi/storcli/storcli /call show termlog > /opt/lsi/storcli/termlog.txt"
"/opt/lsi/storcli/storcli /call/bbu show all > /opt/lsi/storcli/bbu_show.txt"
"/opt/lsi/storcli/storcli /call/cv show all > /opt/lsi/storcli/cvshow.txt"
"/opt/lsi/storcli/storcli /call/dall show all > /opt/lsi/storcli/dallshow.txt"
"/opt/lsi/storcli/storcli /call/eall show all > /opt/lsi/storcli/eallshow.txt"
"/opt/lsi/storcli/storcli /call/eall/sall show all > /opt/lsi/storcli/sallshow.txt"
"/opt/lsi/storcli/storcli /call/vall show all > /opt/lsi/storcli/vallshow.txt"
    
)

foreach ($command in $commands) {
    $result = Invoke-SSHCommand -SessionId $sshSession2.SessionId -Command $command
    #Write-Host $result.Output 
}
Remove-SSHSession -SSHSession $sshSession2 | Out-Null
}

else
{
Write-Host "invalid input"
}

$server = $hostname -replace "\.wal-mart\.com$", ""

mkdir "C:\temp\Storcliscript\$server" -Force | Out-Null



$RemotePath = "/opt/lsi/storcli"  # Path to all .txt files on ESXi
$LocalPath = "C:\temp\storcliscript\$server"  # Local destination folder
$folderPath = "C:\temp\storcliscript\$server\storcli"

# Establish an SSH session
$SFTPSession = New-SFTPSession -ComputerName $hostname -Credential $cred -AcceptKey

# Copy all .txt files from the remote path to the local folder
Get-SFTPItem -SessionId $SFTPSession.SessionId -Path $RemotePath -Destination $LocalPath | Where-Object { $_.Name -like "*.txt" }

# Remove the SFTP session
Remove-SFTPSession -SessionId $SFTPSession.SessionId | Out-Null


#Get all files except .txt files and delete them
Get-ChildItem -Path $folderPath -File | Where-Object { $_.Extension -ne ".txt" } | Remove-Item -Force

#delete the files from esxi

$sshSession3 = New-SSHSession -ComputerName $hostname -Credential $cred -AcceptKey

$commands = @(
"rm /opt/lsi/storcli/showall.txt" 
"rm /opt/lsi/storcli/events.txt"
"rm /opt/lsi/storcli/termlog.txt"
"rm /opt/lsi/storcli/bbu_show.txt"
"rm /opt/lsi/storcli/cvshow.txt"
"rm /opt/lsi/storcli/dallshow.txt"
"rm /opt/lsi/storcli/eallshow.txt"
"rm /opt/lsi/storcli/sallshow.txt"
"rm /opt/lsi/storcli/vallshow.txt"
     
)

foreach ($command in $commands) {
    $result = Invoke-SSHCommand -SessionId $sshSession3.SessionId -Command $command
    #Write-Host $result.Output 
}
Remove-SSHSession -SSHSession $sshSession3 | Out-Null

Write-host "`nStorcli logs collected and saved under C:\temp\Storcliscript\$server`n"

}
