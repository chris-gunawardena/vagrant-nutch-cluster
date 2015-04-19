# -*- mode: ruby -*-
# vi: set ft=ruby :

NODE_COUNT = 3

Vagrant.configure(2) do |config|

	#virtualbox  settings
	config.vm.provider "virtualbox" do |vb|
		vb.memory = "4096"
	end

	NODE_COUNT.times do |i|

		node_id = "node#{i}"
		config.ssh.insert_key = false
				
		config.vm.define node_id do |node|
			#base box
			node.vm.box = "ubuntu/precise64"
			node.vm.box_check_update = false
			node.vm.network "public_network", bridge: 'en0: Wi-Fi (AirPort)', ip: "192.168.0.20#{i}"
			node.vm.hostname = "hadoopnode#{i}"

			#set timezone
			node.vm.provision "shell", inline: "ln -sf /usr/share/zoneinfo/Australia/Melbourne  /etc/localtime"

			#setup hosts file entries for fqdns
			node.vm.provision "shell", inline: " echo \"#{}127.0.0.1 localhost\" > /etc/hosts "
			NODE_COUNT.times do |j|
				#current machine, so add localhost to this ip
				if i == j
					node.vm.provision "shell", inline: " echo \"192.168.0.20#{j}  hadoopnode#{j}.gunawardena.id.au  hadoopnode#{j} localhost\" >> /etc/hosts "
				else
					node.vm.provision "shell", inline: " echo \"192.168.0.20#{j}  hadoopnode#{j}.gunawardena.id.au  hadoopnode#{j}\" >> /etc/hosts "
				end
			end

			#update packages and install ntp
			node.vm.provision "shell", inline: " apt-get update -q "
			node.vm.provision "shell", inline: " apt-get install -q -y -o Dpkg::Options::=\"--force-confdef\" -o Dpkg::Options::=\"--force-confold\" ntp "

			#Only install ambari & nutch in node0
			if i == 0
				node.vm.provision :ansible do |ansible|
					ansible.playbook = "ambari_playbook.yml"
					ansible.limit = 'all'
					ansible.verbose = "v"
				end
				node.vm.provision :ansible do |ansible|
				  ansible.playbook = "nutch_playbook.yml"
				  ansible.limit = 'all'
				  ansible.verbose = "v"
				end
			end

		end
	end
end


# sudo fallocate -l 2G /swapfile
# sudo chmod 600 /swapfile
# sudo mkswap /swapfile
# sudo swapon /swapfile