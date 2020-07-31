# devenv_ansible
build up development environment by ansible

for centos7

Vagrantfile

    config.vm.provision "file", source: "./ansible/", destination: "/home/vagrant/"

    config.vm.provision "shell", inline: <<-SHELL
      yum -y install ansible --exclude=kernel
      ansible-playbook /home/vagrant/ansible/main.yml
    SHELL

Edit vars/main.yml

