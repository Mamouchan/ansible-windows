# Configure testing environement (Azure)

## Goals:

- Install VSCode
- Create a Windows Server 2022 VM
- Configure Winrm on the VM (We will use port 22 instead of 5986 for winrm)

## Instructions (Wait for the instructor on this part)

1) Download VSCode setup from here https://code.visualstudio.com/download and install it

2) Go to https://portal.azure.com and follow the trainer's instructions to create a new Windows VM

3) Enable Winrm on the VM: 

Files to download on the VM

Go to: 
- https://raw.githubusercontent.com/Mamouchan/ansible-windows/refs/heads/main/labs/tools/winrm/ConfigureWinRM.ps1
- https://raw.githubusercontent.com/Mamouchan/ansible-windows/refs/heads/main/labs/tools/winrm/makecert.exe

4) Run the script `ConfigureWinRM.ps1` on the VM: Open a CMD terminal with admin rights and run the following command:

```bash
powershell -ExecutionPolicy Unrestricted -File ConfigureWinRM.ps1
```

## Result

Now you have a windows-server vm ready to be used by ansible :clap:
