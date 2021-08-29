Vagrant.configure("2") do |config|
  config.proxy.enabled = false
  config.ssh.insert_key = false
  ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'
  CONTROLLERNODES=3
  WORKERNODES=3
  LBNODES=1
  node_name="node"
  N = CONTROLLERNODES+WORKERNODES+LBNODES
  (1..N).each do |node_id|
    if node_id <= N-WORKERNODES-LBNODES
      node_name="controller-#{node_id-1}"
      ip = "10.240.0.1#{node_id-1}"
      memory = "1024"
    elsif node_id > N-WORKERNODES-LBNODES && node_id <= N-LBNODES
      node_name="worker-#{node_id-1-CONTROLLERNODES}"
      ip = "10.240.0.2#{node_id-1-CONTROLLERNODES}"
      memory = "512"
    else
      node_name="loadbalancer-#{node_id-1-CONTROLLERNODES-WORKERNODES}"
      ip = "10.240.0.3#{node_id-1-CONTROLLERNODES-WORKERNODES}"
      memory = "512"
    end
    config.vm.define "#{node_name}" do |node|
      node.vm.box = "generic/ubuntu2004"
      node.vm.provider "libvirt" do |libvirt|
          libvirt.memory = "#{memory}"
          libvirt.cpus = 2
      end
      node.vm.provider "virtualbox" do |vb|
          vb.memory = "#{memory}"
          vb.cpus = 2
      end
#      node.vm.hostname = "#{node_name}"
      node.vm.network "private_network", ip: "#{ip}"
      if node_id == N
        node.vm.provision :ansible do |ansible|
          ansible.limit = "all"
          ansible.playbook = "provision.yaml"
        end
        node.vm.provision :ansible do |ansible|
          ansible.groups = {
            "controller" => ["controller-[0:#{CONTROLLERNODES-1}]"],
            "worker" => ["worker-[0:#{WORKERNODES-1}]"],
            "loadbalancer" => ["loadbalancer-[0:#{LBNODES-1}]"],
          }
          ansible.playbook = "plays/site.yaml"
          ansible.limit = "all"
          ansible.host_key_checking = false
          ansible.verbose =  'v'
          ansible.extra_vars = { ansible_user: 'vagrant',
                 ansible_connection: 'ssh',
                 ansible_ssh_args: '-o ForwardAgent=yes -o UserKnownHostsFile=/dev/null -o AddKeysToAgent=yes'}
        end
      end
    end
  end
end
