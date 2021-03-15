IMAGE_NAME = "centos/7"
N = 1
USERNAME = 'delphix'
PASSWORD = 'delphix'
ENGINE = '172.168.26.9'

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    config.vm.provider "vmware_desktop" do |v|
        v.vmx["memsize"] = 2048
        v.vmx["numvcpus"] = 1
        v.ssh_info_public = true
        # https://github.com/hashicorp/vagrant/issues/12045
        v.gui = true
        v.vmx["ethernet0.pcislotnumber"] = "32"
    end
      
    config.vm.define "postgres-primary" do |primary|
        primary.vm.box = IMAGE_NAME
        #primary.vm.network "public_network", use_dhcp_assigned_default_route: true
        primary.vm.network "private_network", ip: "172.168.26.20"
        primary.vm.hostname = "postgres-primary"
        primary.vm.provision "ansible" do |ansible|
            ansible.playbook = "ansible/primary-playbook.yml"
            ansible.extra_vars = {
                        node_ip: "172.168.26.20",
                        node_name: "postgres-primary",
		        uusername: "#{USERNAME}",
		        upassword: "#{PASSWORD}",
                        engine: "#{ENGINE}",
            }
        end
    end

    (1..N).each do |i|
        config.vm.define "postgres-secondary-#{i}" do |node|
            node.vm.box = IMAGE_NAME
            # primary playbook
            node.vm.network "private_network", ip: "172.168.26.#{i + 20}"
            node.vm.network "forwarded_port", guest: 22, host: 2222, id: "ssh", auto_correct: true
            node.vm.hostname = "postgres-secondary-#{i}"
            node.vm.provision "primary-playbook", type:'ansible' do |ansible|
                ansible.playbook = "ansible/primary-playbook.yml"
                ansible.extra_vars = {
                    node_ip: "172.168.26.#{i + 20}",
                    node_name: "postgres-secondary-#{i}",
                    uusername: "#{USERNAME}",
		            upassword: "#{PASSWORD}",
                    engine: "#{ENGINE}",
                }
            end
        end
    end
end
