# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    
    # if Vagrant.has_plugin?("vagrant-vbguest")
    #     config.vbguest.auto_update = false
    #   end

    config.vm.box = "centos/7"
    config.vm.box_check_update = true
    config.vm.provider "virtualbox" do |vb|
        vb.memory = "1024"
        vb.cpus = "1"
    end

    config.ssh.insert_key = "false"
   
    config.vm.provision "shell", inline: <<-SHELL
        echo "vagrant:vagrant" | chpasswd
        sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/g' /etc/ssh/sshd_config
        sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
        systemctl restart sshd.service
        echo "192.168.10.10 backupserver" >> /etc/hosts
        echo "192.168.10.11 client" >> /etc/hosts
        echo "192.168.10.111 ansiblecontroller" >> /etc/hosts
        SHELL

    config.vm.define "backupserver", primary: true do |backupserver|
            
            backupserver.vm.provider "virtualbox" do |vb|
                unless FileTest.exist?('./backup.vdi')
                    vb.customize ['createhd', '--filename', './backup.vdi', '--size', 2 * 1024]
                end
                vb.customize ['storageattach', :id, '--storagectl', 'IDE', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', './backup.vdi']
                
            end
            backupserver.vm.hostname = "backupserver"
            backupserver.vm.network "private_network", ip: "192.168.10.10", virtualbox__intnet: true, name: "provision_net"
        end

    config.vm.define "client", primary: true do |client|
            client.vm.hostname = "client"
            client.vm.network "private_network", ip: "192.168.10.11", virtualbox__intnet: true, name: "provision_net"
        end

    config.vm.define "ansiblecontroller", primary: true do |ansiblecontroller|
            ansiblecontroller.vm.network "private_network", ip: "192.168.10.111", virtualbox__intnet: true, name: "provision_net"
            ansiblecontroller.vm.hostname = "ansiblecontroller"
            #ansiblecontroller.vm.synced_folder ".", "/vagrant", type: "smb" # for windows
                ansiblecontroller.vm.provision "ansible", run: "always", type: "ansible_local"  do |ansible|
                    ansible.playbook       = "provisioning/playbook.yml"
                    ansible.install        = true
                    #ansible.verbose        = true
                    ansible.limit          = "all"
                    ansible.inventory_path = "hosts"
                    #ansible.galaxy_role_file = "provisioning/requirements.yml"
                    #ansible.galaxy_roles_path = "/etc/ansible/collections"
                    #ansible.galaxy_command = "sudo ansible-galaxy collection install -r %{role_file} -p %{roles_path} --force"
                end
        end      
end    