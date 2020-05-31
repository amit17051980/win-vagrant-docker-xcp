# Setup Docker xCP Environment using Vagrant

Setup Documentum xCP docker environment using Vagrant and Docker Container on Hyper-V VM. The guide below is valid for Windows 10 Pro users who has Administrator permissions to run referenced commands.

## Host Environment

| Host OS        	| Virtualization Software 	| Guest OS 	| Guest Container 	|
|----------------	|-------------------------	|----------	|-----------------	|
| Windows 10 Pro 	| Hyper-V                 	| CentOS 7 	| Docker          	|

## Install Windows package manager (Chocolatey) and Hyper-V

Open the Windows PowerShell as Administrator, and run the commands below.
```PowerShell
cd ~
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1')) 
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

## Install Git and Vagrant using Chocolatey

Open the Windows PowerShell as Administrator, and run the commands below.
```PowerShell
cd ~
choco install -y git # Remove if you already have git
choco install -y vagrant # Remove if you already have vagrant
```

## Clone the Project

This step integrate many manual steps into one Vagrantfile and delivers the Dockerised xCP environment. Some command-line options will be presented to deal with Hyper-V VM provisioning, and until the limitations are removed, will be not automated. Open a new Windows PowerShell as Administrator, and run the commands below.

```PowerShell
cd ~
mkdir DCTM-Projects
cd DCTM-Projects
git clone https://github.com/amit17051980/win-vagrant-docker-xcp.git
cd .\win-vagrant-docker-xcp
```

Copy the below files into '.\win-vagrant-docker-xcp\media-files'

