# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"
  config.ssh.forward_agent = true
  config.ssh.insert_key = false
  config.hostmanager.enabled = true
  config.cache.scope = :box

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.synced_folder_opts = {
      owner: "_apt"
    }
  end
 
  # We need one Ceph admin machine to manage the cluster
  config.vm.define "ceph-admin" do |admin|
    admin.vm.hostname = "ceph-admin"
    admin.vm.network :private_network, ip: "172.21.12.10"
    admin.vm.provision :shell, :inline => "DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -yq ntp && apt-get install -yq build-essential gcc make perl dkms", :privileged => true
    admin.vm.provision :shell, :inline => "DEBIAN_FRONTEND=noninteractive wget -q -O- 'https://download.ceph.com/keys/release.asc' | apt-key add - && deb https://download.ceph.com/debian-luminous/ $(lsb_release -sc) main | tee /etc/apt/sources.list.d/ceph.list && apt update -yq && apt install -yq ceph-deploy", :privileged => true
    # add optical drive for Guest Additions
    admin.vm.provider "virtualbox" do |vb|
      vb.customize ["storageattach", :id,
                    "--storagectl", "IDE",
                    "--port", "0", "--device", "1",
                    "--type", "dvddrive",
                    "--medium", "emptydrive"]
      end
  end

  # The Ceph client will be our client machine to mount volumes and interact with the cluster
  config.vm.define "ceph-client" do |client|
    client.vm.hostname = "ceph-client"
    client.vm.network :private_network, ip: "172.21.12.11"
    # ceph-deploy will assume remote machines have python2 installed
    client.vm.provision :shell, :inline => "DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -yq python && apt-get install -yq build-essential gcc make perl dkms", :privileged => true
        
    # add optical drive for Guest Additions
    client.vm.provider "virtualbox" do |vb|
      vb.customize ["storageattach", :id,
                    "--storagectl", "IDE",
                    "--port", "0", "--device", "1",
                    "--type", "dvddrive",
                    "--medium", "emptydrive"]
      end
  end

  # We provision three nodes to be Ceph servers
  (1..3).each do |i|
    config.vm.define "ceph-server-#{i}" do |config|
      config.vm.hostname = "ceph-server-#{i}"
      config.vm.network :private_network, ip: "172.21.12.#{i+11}"
      config.vm.provider "virtualbox" do |v|
        filename="./.vagrant/ceph#{i}.disk"
        #This next step can fail if the disk is already created, eg if you
        #CTRL+C'd a previous vagrant up, so wrap it in conditional.
        unless File.exist?(filename)
          #Create a sparse volume with a max size of 10GB called
          v.customize ['createhd', '--filename', filename, '--size', 10 * 1024]
        end
        #You may have to tweak the value for --storagectl.
        #Other folks have noted what worked for them here:
        # https://gist.github.com/leifg/4713995
        v.customize ['storageattach', :id, '--storagectl', 'SCSI', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', filename]
        end
        
      # ceph-deploy will assume remote machines have python2 installed
      config.vm.provision :shell, :inline => "DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -yq python && apt-get install -yq build-essential gcc make perl dkms", :privileged => true
          
    # add optical drive for Guest Additions
    config.vm.provider "virtualbox" do |vb|
      vb.customize ["storageattach", :id,
                    "--storagectl", "IDE",
                    "--port", "0", "--device", "1",
                    "--type", "dvddrive",
                    "--medium", "emptydrive"]
      end
    end
  end
end