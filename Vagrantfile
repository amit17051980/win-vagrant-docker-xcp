# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "centos/7"
  config.vm.hostname = "documentum-xcp"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "WIFI Network"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder ".", "/dctm", create: true

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "hyperv" do |hv|
    # Display the VirtualBox GUI when booting the machine
    # hv.gui = true
    #   # Customize the amount of memory on the VM:
	hv.vmname = "documentum-xcp"
    hv.maxmemory = "8000"
	hv.cpus = "2"
	hv.enable_virtualization_extensions = true
	hv.vm_integration_services = {
		guest_service_interface: true #<---------- this line enables Copy-VMFile
		# https://techcommunity.microsoft.com/t5/virtualization/copying-files-into-a-hyper-v-vm-with-vagrant/ba-p/382376
		# Copy-VMFile -Name documentum-xcp -SourcePath 'C:\Users\amit_\Downloads\ContentServer_16.4.0140_Docker_Centos.tar' -DestinationPath '/tmp' -FileSource Host
	}
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", env: {"DHUBID"=>ENV['DHUBID'], "DHUBPASS"=>ENV['DHUBPASS']}, inline: <<-SHELL
    yum -y update
	yum -y install -y yum-utils
	yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
	yum -y install docker-ce docker-ce-cli containerd.io
	systemctl enable docker
	systemctl start docker
	usermod -aG docker vagrant
	sleep 5s
	docker ps
	docker login --username $DHUBID --password $DHUBPASS 
	docker pull $DHUBID/dctm-cs:16.4.0
	
	# Install Docker Composer
	sudo curl -L \
		"https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" \
		-o /usr/local/bin/docker-compose
	sudo chmod +x /usr/local/bin/docker-compose
	sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
	docker-compose --version
	
	# Create Docker Network
	docker network create dctm-dev
	# Create and configure Postgres Container for Documentum repository
	docker run --network dctm-dev --name postgres --hostname postgres -e POSTGRES_PASSWORD=password -d -p 5432:5432 postgres:9.6
	sleep 5s
	docker exec -it postgres su -c "mkdir /var/lib/postgresql/data/db_documentum_dat.dat" postgres
	
	# Setup documentum content server with repository and method server
	source /dctm/documentum-environment.profile
    docker-compose -f /dctm/CS-Docker-Compose_Stateless.yml up -d
	
	# Setup Tomcat for DA and xCP Apphost
	docker run --network dctm-dev -d --name documentum-da --hostname documentum-da -p 8080:8080 amit17051980/dctm-tomcat:latest
	docker exec -it documentum-da su -c "mv /usr/local/tomcat/webapps.dist/manager /usr/local/tomcat/webapps/"
	docker cp /dctm/media-files/tomcat-users.xml documentum-da:/usr/local/tomcat/conf/
	docker cp /dctm/media-files/context.xml documentum-da:/usr/local/tomcat/webapps/manager/META-INF/
	
	
	docker restart documentum-da
	
	# Deploy DA and Setup Custom-Conf
	
	echo "All DONE!"
  SHELL
end
