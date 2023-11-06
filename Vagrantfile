NUM_WORKER_NODES=2
IP_NW="172.31.0."
IP_START=100

Vagrant.configure("2") do |config|
  config.vm.provision "shell", inline: "dnf -y upgrade"
  config.vm.box = "bento/centos-stream-8"
  config.vm.box_check_update = false
  config.ssh.insert_key = "true"
  config.vm.synced_folder '.', '/vagrant', disabled: true

  config.vm.define "master" do |master|
    master.vm.hostname = "master-node"
    master.vm.network "private_network", ip: IP_NW + "#{IP_START}", hostname: true
    master.vm.network "forwarded_port", guest: 22, host: 2220, id: "ssh"
    master.vm.provider "vmware_desktop" do |vb|
      vb.vmx["memsize"] = "4096"
      vb.vmx["numvcpus"] = "2"
      vb.vmx["name"] = "k8s-centos-master-node"
    end
  end

  (1..NUM_WORKER_NODES).each do |i|
    config.vm.define "node-#{i}" do |node|
      node.vm.hostname = "worker-node-#{i}"
      node.vm.network "private_network", ip: IP_NW + "#{IP_START + i}", hostname: true
      node.vm.network "forwarded_port", guest: 22, host: "222#{i}", id: "ssh"
      node.vm.provider "vmware_desktop" do |vb|
        vb.vmx["memsize"] = "2048"
        vb.vmx["numvcpus"] = "2"
        vb.vmx["name"] = "k8s-centos-worker-node-#{i}"
      end
      # Only execute once the Ansible provisioner,
      # when all the machines are up and ready.
      if i == NUM_WORKER_NODES
        node.vm.provision "ansible" do |ansible|
          # Disable default limit to connect to all the machines
          ansible.config_file = "ansible.cfg"
          ansible.inventory_path = "inventory"
          ansible.limit = "all"
          ansible.playbook = "k8s.yml"
          ansible.compatibility_mode = "2.0"
        end
      end
    end
  end
end
