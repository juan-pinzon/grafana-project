# -*- mode: ruby -*-
# vi: set ft=ruby :

$scriptInit = <<SCRIPT
apt-get remove docker docker-engine docker.io containerd runc
apt-get update
apt-get install -y apt-transport-https && apt-get -y install apt-transport-https 
apt-get install -y ca-certificates && apt-get install -y curl
apt-get install -y gnupg-agent && apt-get install -y software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

apt-get update
apt-get install -y docker-ce docker-ce-cli containerd.io

curl -L https://github.com/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
SCRIPT


Vagrant.configure("2") do |config|
	if Vagrant.has_plugin? "vagrant-vbguest"
		config.vbguest.no_install = true
		config.vbguest.auto_update = false
		config.vbguest.no_remote = true
	end
	
	config.vm.define :server do |server|
      server.vm.box = "bento/ubuntu-20.04"
      server.vm.network :private_network, ip: "192.168.57.2"
      server.vm.hostname = "server"
	  server.vm.provision :shell, inline: $scriptInit
	end

	config.vm.define :client do |client|
		client.vm.box = "bento/ubuntu-20.04"
		client.vm.network :private_network, ip: "192.168.57.3"
		client.vm.hostname = "client"
		client.vm.provision :shell, inline: $scriptInit
	end
end