# Setup Documentum xCP Docker Environment using Vagrant on Windows 10

Setup Documentum xCP docker environment using Vagrant and Docker Container on Hyper-V VM. The guide below is valid for Windows 10 Pro users who has Admin permissions to run referenced commands.

## Assumptions

Below assumtions have been made while creating this document. Please skip the relevant sections below if you already have referenced components installed/configured.

* Windows package manager (Chocolatey) and Hyper-V are not installed and configured
* Git and Vagrant are not installed and configured
* A private docker registry exists, and is accessible to use documentum content server image 

## Host Environment

| Configuration        	| Software/Specs 	          | Other Info                              |
|----------------	      |-------------------------	|-----------                              |
| Windows 10 Pro (HOST) | Hyper-V                 	| CentOS 7 (GUEST), Docker (CONTAINER) 	  |
| RAM                   | 8 of 16 GB is available   |                                         |
| CORES                 | 2 of 6 are available      |                                         |
| DISK                  | 50 GB SSD available       |                                         |

## (Optional) Install Windows package manager (Chocolatey) and Hyper-V

Open the Windows PowerShell as Administrator, and run the commands below.
```PowerShell
cd ~
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1')) 
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

## (Optional) Install Git and Vagrant using Chocolatey

Open the Windows PowerShell as Administrator, and run the commands below.
```PowerShell
cd ~
choco install -y git # Remove if you already have git
choco install -y vagrant # Remove if you already have vagrant
```

## Create a Project directory and Clone the Project in <i>$HOME/DCTM-Projects</i>

This step integrate many manual steps into one Vagrantfile and delivers the Dockerised xCP environment. Some command-line options will be presented to deal with Hyper-V VM provisioning, and until the limitations are removed, will be not automated. Open <b><i>a new</i></b> Windows PowerShell as Administrator, and run the commands below.

```PowerShell
cd ~
mkdir DCTM-Projects
cd DCTM-Projects
git clone https://github.com/amit17051980/win-vagrant-docker-xcp.git
cd .\win-vagrant-docker-xcp
```

## Copy the files below into '.\win-vagrant-docker-xcp\media-files'

* TBC
* TBC

## Update the files below before proceed

* media-files/tomcat-users.xml : Update passwords
* documentum-environment.profile : Update passwords

## Run the command below to create the VM with all the relevant containers.

```PowerShell
# Import the environment variables to connect with private docker hub registry. 
# This is not a good practice, and would highly recommend to use https://docs.docker.com/engine/reference/commandline/login/
# In this instance, we are enforcing to get rid of history at the end of session exit.

Set-PSReadlineOption -HistorySaveStyle SaveNothing
$env:DHUBID = 'Docker HUB ID'
$env:DHUBPASS = 'Docker HUB PASSWORD' 

cd ~\DCTM-Projects\win-vagrant-docker-xcp
vagrant up
```



