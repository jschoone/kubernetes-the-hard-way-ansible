Vagrant.configure("2") do |config|
  config.proxy.enabled = false
  ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'
  CONTROLLERNODES=3
  WORKERNODES=3
  LB=1
  node_name="node"
  memory="512"
  N = CONTROLLERNODES+WORKERNODES+LB
  (1..N).each do |node_id|
    if node_id <= N-WORKERNODES-LB
      node_name="controller-#{node_id-1}"
      ip = "10.240.0.1#{node_id-1}"
      memory = "1024"
    elsif node_id > N-WORKERNODES-LB && node_id <= N-LB
      node_name="worker-#{node_id-1-CONTROLLERNODES}"
      ip = "10.240.0.2#{node_id-1-CONTROLLERNODES}"
    else
      node_name="loadbalancer-#{node_id-1-CONTROLLERNODES-WORKERNODES}"
      ip = "10.240.0.3#{node_id-1-CONTROLLERNODES-WORKERNODES}"
    end
    config.vm.define "#{node_name}" do |node|
      node.vm.box = "generic/ubuntu2004"
      node.vm.provider "libvirt" do |libvirt|
          libvirt.memory = "#{memory}"
          libvirt.cpus = 2
      end
      vm_config.vm.provider "virtualbox" do |vb|
          vb.memory = "#{memory}"
          vb.cpus = 2
      end
#      node.vm.hostname = "#{node_name}"
      node.vm.network "private_network", ip: "#{ip}"
      if node_id == N
        node.vm.provision :ansible do |ansible|
          ansible.limit = "all"
          ansible.playbook = "ansible/provision.yaml"
        end
      end
    end
  end
end
