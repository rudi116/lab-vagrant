# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.box_version = "20200206.0.0"
  config.vm.hostname = "devops"

  config.vm.network "forwarded_port", guest: 5000, host: 5000, host_ip: "127.0.0.1"
  config.vm.network "private_network", ip: "192.168.33.10"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "512"
    vb.cpus = 1

    # Fixes some DNS issues on some networks
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
  end

  ############################################################
  # Copy some host files to configure VM like the host
  ############################################################

  # Copy your .gitconfig file so that your git credentials are correct
  if File.exists?(File.expand_path("~/.gitconfig"))
    config.vm.provision "file", source: "~/.gitconfig", destination: "~/.gitconfig"
  end

  # Copy your ssh keys for github so that your git credentials work
  if File.exists?(File.expand_path("~/.ssh/id_rsa"))
    config.vm.provision "file", source: "~/.ssh/id_rsa", destination: "~/.ssh/id_rsa"
  end
  if File.exists?(File.expand_path("~/.ssh/id_rsa.pub"))
    config.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "~/.ssh/id_rsa.pub"
  end

  # Copy your .vimrc file so that your VI editor looks nice
  if File.exists?(File.expand_path("~/.vimrc"))
    config.vm.provision "file", source: "~/.vimrc", destination: "~/.vimrc"
  end

  ############################################################
  # Create a Python 3 environment for development work
  ############################################################
  config.vm.provision "shell", inline: <<-SHELL
    # Update and install
    apt-get update
    apt-get install -y git tree python3-dev python3-pip python3-venv apt-transport-https
    apt-get upgrade python3
    apt-get -y autoremove

    # Create a Python3 Virtual Environment and Activate it in .profile
    sudo -H -u vagrant sh -c 'python3 -m venv ~/venv'
    sudo -H -u vagrant sh -c 'echo ". ~/venv/bin/activate" >> ~/.profile'
    sudo -H -u vagrant sh -c '. ~/venv/bin/activate && cd /vagrant && pip install -r requirements.txt'

    # Install app dependencies
    # cd /vagrant
    # pip3 install -r requirements.txt
  SHELL

  ######################################################################
  # Add Redis docker container
  ######################################################################
  config.vm.provision "shell", inline: <<-SHELL
    # Prepare Redis data share
    sudo mkdir -p /var/lib/redis/data
    sudo chown vagrant:vagrant /var/lib/redis/data
  SHELL

  # Add Redis docker container
  config.vm.provision "docker" do |d|
    d.pull_images "redis:alpine"
    d.run "redis",
      image: "redis:alpine",
      args: "-h redis -p 6379:6379 -v /var/lib/redis/data:/data",
      restart: "always",
      daemonize: true
  end

end
