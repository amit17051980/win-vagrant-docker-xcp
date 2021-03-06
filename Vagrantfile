Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.hostname = "documentum-xcp"
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "public_network"
  config.vm.synced_folder ".", "/dctm", create: true

  config.vm.provider "hyperv" do |hv|
	hv.vmname = "documentum-xcp"
	hv.maxmemory = "8000"
	hv.cpus = "2"
	hv.enable_virtualization_extensions = true
	hv.vm_integration_services = {
		#<---------- this line enables Copy-VMFile
		# https://techcommunity.microsoft.com/t5/virtualization/copying-files-into-a-hyper-v-vm-with-vagrant/ba-p/382376
		# Copy-VMFile -Name documentum-xcp -SourcePath 'C:\Users\amit_\Downloads\ContentServer_16.4.0140_Docker_Centos.tar' -DestinationPath '/tmp' -FileSource Host
		guest_service_interface: true 
	}
  end

  config.vm.provision "shell", env: {"DHUBID"=>ENV['DHUBID'], "DHUBPASS"=>ENV['DHUBPASS']}, inline: <<-SHELL
	yum -y update
	yum -y install -y yum-utils
	yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
	yum -y install docker-ce docker-ce-cli containerd.io
	systemctl enable docker
	systemctl start docker
	usermod -aG docker vagrant
	
	# Configuring Docker Hub Private Registry
	docker login --username $DHUBID --password $DHUBPASS 
	docker pull $DHUBID/dctm-cs:16.4.0
	
	# Install Docker Composer
	FILE=/usr/bin/docker-compose
	if [ ! -f "$FILE" ]; then
		sudo curl -L \
		"https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" \
		-o /usr/local/bin/docker-compose
		sudo chmod +x /usr/local/bin/docker-compose
		sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
	fi
	
	# Install DB, Documentum
	source /dctm/documentum-environment.profile
	
	DIR=/var/lib/docker/volumes/dctm_documentum_dba/_data/config/documentum
	if [ ! -d "$DIR" ]; then
	
		docker-compose --version
	
		# Create Docker Network
		docker network create dctm-dev
		# Create and configure Postgres Container for Documentum repository
		docker run --network dctm-dev --name postgres --hostname postgres -e POSTGRES_PASSWORD=password -d -p 5432:5432 postgres:9.6
		sleep 5s
		docker restart postgres
		sleep 5s
		docker exec postgres su -c "mkdir /var/lib/postgresql/data/db_documentum_dat.dat" postgres
		
		# Setup documentum content server with repository and method server
		docker-compose -f /dctm/CS-Docker-Compose_Stateless.yml up -d
		
		# Fixing the restart of documentum docbase post reprovisioning
		# The password has to be same as $AEK_PASSPHRASE
		
		# sleep 5s
		# echo "AEK_PASSPHRASE : ${AEK_PASSPHRASE}"
		# docker exec documentum-cs su - dmadmin -c "dm_crypto_boot -all -passphrase ${AEK_PASSPHRASE} -noprompt"
		# sleep 5s
		# docker exec documentum-cs su - dmadmin -c "/opt/dctm/dba/dm_start_documentum"
	else
		docker start postgres
		sleep 5s
		docker start documentum-cs
		sleep 5s
		docker exec documentum-cs su - dmadmin -c "export AEK_PASSPHRASE=${AEK_PASSPHRASE} && dm_crypto_boot -all -passphrase $AEK_PASSPHRASE"
		docker exec documentum-cs su - dmadmin -c "/opt/dctm/dba/dm_start_documentum"
	fi 
	
	
	# Setup Tomcat for DA and xCP Apphost
	docker run --network dctm-dev -d --name documentum-da --hostname documentum-da -p 8080:8080 amit17051980/dctm-tomcat:latest
	sleep 5s
	# docker cp documentum-da:/usr/local/tomcat/webapps.dist/manager /dctm/media-files/
	# docker cp /dctm/media-files/manager documentum-da:/usr/local/tomcat/webapps/
	docker exec documentum-da su -c "cp -r /usr/local/tomcat/webapps.dist/manager /usr/local/tomcat/webapps/"
	docker cp /dctm/media-files/tomcat-users.xml documentum-da:/usr/local/tomcat/conf/
	docker cp /dctm/media-files/context.xml documentum-da:/usr/local/tomcat/webapps/manager/META-INF/
	
	
	sleep 5s
	docker restart documentum-da
	
	# Deploy DA and Setup Custom-Conf
	
	
	docker ps
	# Clear Passwords from session and history
	# unset APP_SERVER_PASSWORD INSTALL_OWNER_PASSWORD ROOT_PASSWORD DOCBASE_PASSWORD DATABASE_PASSWORD GLOBAL_REGISTRY_PASSWORD AEK_PASSPHRASE LOCKBOX_PASSPHRASE
	
	echo "All DONE!"
  SHELL
end
