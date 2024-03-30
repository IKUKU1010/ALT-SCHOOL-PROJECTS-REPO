# ALT SCHOOL CLOUD ENGINEERING LIVE CLASS ASSIGNMENT 3

The Assignment tasks requires the provisioning of three VM's one Master node withy two slave nodes. Then create and Ansible playbook that will install apache server on the slave nodes from the master machine and depoly a website or web app on the apache servers.

### WE first create a vagrantfile that will deploy these servers concurrently and configured to be in same netwok.

The vagrantfile for this is configured as shown below

```bash

# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  # Ubuntu VM
  config.vm.define "ubuntu" do |ubuntu|
     ubuntu.vm.box = "ubuntu/trusty64"
     ubuntu.vm.hostname = "ubuntu-slave"
     ubuntu.vm.network "private_network", ip: "192.168.56.24"
     ubuntu.vm.network "forwarded_port", guest: 80, host: 8082
   end

 
   # Debian 10 VM
   config.vm.define "debian" do |debian|
     debian.vm.box = "debian/buster64"
     debian.vm.hostname = "debian-slave"
     debian.vm.network "private_network", ip: "192.168.56.22"
     debian.vm.network "forwarded_port", guest: 80, host: 8081
   end
 
   # Ubuntu 20.04 VM
   config.vm.define "master" do |master|
     master.vm.box = "ubuntu/focal64"
     master.vm.hostname = "master-node"
     master.vm.network "private_network", ip: "192.168.56.16"
     master.vm.network "forwarded_port", guest: 80, host: 8080
   end

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Disable the default share of the current code directory. Doing this
  # provides improved isolation between the vagrant box and your host
  # by making sure your Vagrantfile isn't accessible to the vagrant box.
  # If you use this you may want to enable additional shared subfolders as
  # shown above.
  # config.vm.synced_folder ".", "/vagrant", disabled: true

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end

```

![vagrant up/ servers up and Running](./Servers%20up%20and%20Running.png)

---


We now login into the master machine by running

```bash

vagarant ssh master

```

We then install ansible on the master server by running the following commands

```bash

sudo apt update
sudo apt install ansible

```

We then install vagrant host manager

```bash

sudo apt install vagrant-hostmanager


```

## download our custom webpage/website via git. save the custom website on the vagrant folder

git clone https://github.com/IKUKU1010/EmmanuelBodata.git





## We now Create an Ansible Playbook to automate the installation of our webservers on the slave nodes

We will intall apche webserver on the ubuntu slave and nginx on the debian

```bash

---
- name: Install web server
  hosts: all
  become: true
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: install nginx
      apt:
        name: nginx
        state: latest
      when: ansible_distribution == 'Debian'
    - name: start nginx
      service:
        name: nginx
        state: started
        enabled: true 
      when: ansible_distribution == 'Debian'
    - name: restart nginx
      service:
        name: nginx
        state: restarted
      when: ansible_distribution == 'Debian'

    - name: Install Apache (Ubuntu)
      apt:
        name: apache2
        state: present
      when: ansible_distribution == 'Ubuntu'

    - name: start apache2
      service:
        name: apache2
        state: started
        enabled: true 
      when: ansible_distribution == 'Ubuntu'

    - name: restart apache2
      service:
        name: apache2
        state: restarted
      when: ansible_distribution == 'Ubuntu'


    - name: Copy custom webpage to Apache document root
      copy:
        src: /home/vagrant/.ansible/EmmanuelBodata/ 
        dest: /var/www/html/index.html
      when: ansible_distribution == 'Ubuntu'
        owner: root
        group: root
        mode: '0644'

    - name: Copy custom webpage to Nginx document root
      copy:
        src: /home/vagrant/.ansible/EmmanuelBodata/   
        dest: /var/www/html/index.html 
      when: ansible_distribution == 'Debian'
        owner: root
        group: root
        mode: '0644'


    - name: Disable default Apache page
      file:
        path: /var/www/html/index.html
        state: absent
      when: ansible_distribution == 'Ubuntu'

    - name: Disable default Nginx page
      file:
        path: /var/www/html/index.html
        state: absent
      when: ansible_distribution == 'Debian'



```

## Update the provisioner block in your Vagrantfile to point to the modified playbook:

```bash

config.vm.provision "ansible" do |ansible|
  ansible.playbook = "install_web_playbook.yml"
  ansible.inventory_path = ".vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory"
end


```

Restart our Vagrant environment as usual with vagrant up.

With this setup, the Ansible playbook will install the appropriate web server (Apache or Nginx) based on the operating system of the target node