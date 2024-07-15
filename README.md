# Setup
Setup workstation from command line.
Overview:
1. [Windows](#windows)
1. [VM Linux](#vm-linux)
    1. [Forward SSH access on Windows](#forward-ssh-access-on-windows)
1. [Git](#git)
1. [Python](#python)
1. [VSCode](#vscode)
1. [References](#references)

# Windows
```powershell
# powershell
# install git, powershell windows terminal, vscode
winget install --id Git.Git -e --source winget
winget install --id Microsoft.Powershell --source winget
winget install -e --id Microsoft.WindowsTerminal
winget install -e --id Microsoft.VisualStudioCode
# optional: obsidian
winget install -e --id Obsidian.Obsidian
```

# VM Linux
From Windows Powershell:
```powershell
$debianISO = "$env:USERPROFILE\Downloads\debian.iso"
$vbox = "C:\Program Files\Oracle\VirtualBox"
$machine = "Debian"
$dest = "$env:USERPROFILE\vm" # path to VM ($machine will be applied)
$destDrive = "$dest\$machine\${machine}_disk.vdi"
$driveSize = 256000 # 250gb
$ramSize = 16384 # 16gb
$portSSH = 22
$CPUs = 8

# do it once
winget install -e --id Oracle.VirtualBox
curl -kLo $debianISO  https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.5.0-amd64-netinst.iso

# create and configure VM
& $vbox\VBoxManage createvm --name $machine --ostype "Debian_64" --register --basefolder $dest
& $vbox\VBoxManage modifyvm $machine --cpus $CPUs --memory $ramSize --vram 20 --graphicscontroller=vmsvga --ioapic on --nic1 nat --natpf1 "guestssh,tcp,,$portSSH,,22"
& $vbox\VBoxManage createhd --filename $destDrive --size $driveSize --format VDI --variant Fixed
& $vbox\VBoxManage storagectl $machine --name "SATA Controller" --add sata --controller IntelAhci
& $vbox\VBoxManage storageattach $machine --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium $destDrive
& $vbox\VBoxManage storagectl $machine --name "IDE Controller" --add ide --controller PIIX4
& $vbox\VBoxManage storageattach $machine --storagectl "IDE Controller" --port 1 --device 0 --type dvddrive --medium $debianISO

# start
& $vbox\VBoxManage startvm $machine
# optional:
# add alias "deb-start"
"function deb-start { & '`$vbox\VBoxManage' startvm `$machine }" >> $profile
. $profile # restart shell
```
Minimal SSH server:
1. `Graphical Install` -> `English` -> `United States` -> `American English`
1. `debian` -> empty domain -> root password -> user name -> user password
1. `Eastern` -> `Guided - use entire disk` -> `SCSI3 (0,0,0) (sda)`
1. `All files in one partition` -> `Finish partitioning and write changes` -> `Yes`
1. No scan -> `United states` -> `deb.debian.org` -> empty HTTP proxy -> don't participate
1. **Select only**: `SSH server`, `Standard system utilities`
1. `Yes` -> `/dev/sda` -> `Continue`

## Forward SSH access on Windows:
```powershell
# powershell
ssh-keygen -t ed25519 -b 4096 # select default (empty) fields
$userAtHost = "user@localhost"
$keyPath = "$env:USERPROFILE\.ssh\id_ed25519.pub"

$key=(Get-Content "$keyPath" | Out-String)
ssh $userAtHost "mkdir -p ~/.ssh && chmod 700 ~/.ssh && echo '${key}' >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"

# create config. Access will be: "ssh deb"
$privateKey = $keyPath.replace(".pub", "")
$config = @"
Host deb
 User user
 HostName localhost
 Port 22
 IdentityFile $privateKey
"@
$config > .ssh\config
ssh deb # check
```

# Git
```powershell
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
git config --global core.autocrlf true
git config --global init.defaultbranch main
```

# Python
```powershell
winget install -e python3
# update your path
"`$env:PATH = `"`$env:LocalAppdata\Programs\Python\Python312;`$env:PATH`"" >> $profile
. $profile # restart shell
# open powershell as admin or enable developer mode (windows search: developer settings):
cd $env:LocalAppdata\Programs\Python\Python312
cmd /c mklink python310.dll python312.dll
cmd /c mklink python3.exe python.exe
# windows search: "Manage app aliases" -> disable python and python3 installers
# restart powershell session
```

# VSCode
Press `Ctrl+Shift+P`: `Open User Settings (JSON)`:
```json
{
    "editor.fontFamily": "Cascadia Code",
    "git.alwaysSignOff": true,
    "diffEditor.ignoreTrimWhitespace": false,
    "editor.renderFinalNewline": "on",
    "editor.trimAutoWhitespace": true,
    "editor.renderWhitespace": "trailing",
    "files.trimTrailingWhitespace": true,
    "files.insertFinalNewline": true,
    "files.trimFinalNewlines": true,
    "C_Cpp.clang_format_fallbackStyle": "LLVM",
    "C_Cpp.clang_format_style": "LLVM",
    "[python]": {
        "editor.defaultFormatter": "ms-python.autopep8",
    }
}
```

# References
- [How to create a VirtualBox VM from command line | Andrea Fortuna](https://andreafortuna.org/2019/10/24/how-to-create-a-virtualbox-vm-from-command-line/)
- [Visual Studio Code Remote Development Troubleshooting Tips and Tricks](https://code.visualstudio.com/docs/remote/troubleshooting#_quick-start-using-ssh-keys)
- [How to: Create a PowerShell alias with parameters - SeanKilleen.com](https://seankilleen.com/2020/04/how-to-create-a-powershell-alias-with-parameters/)
- [Git - First-Time Git Setup (git-scm.com)](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup)
