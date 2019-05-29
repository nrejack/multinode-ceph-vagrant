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
    admin.vm.provision :shell, :inline => "DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -yq ntp && apt-get install build-essential gcc make perl dkms", :privileged => true
    
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
    config.vm.provision :shell, :inline => "DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -yq python && apt-get install build-essential gcc make perl dkms", :privileged => true
        
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
      # ceph-deploy will assume remote machines have python2 installed
      config.vm.provision :shell, :inline => "DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -yq python && apt-get install build-essential gcc make perl dkms", :privileged => true
          
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