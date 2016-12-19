# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box     = "ubuntu/trusty64"

  config.vm.provider "virtualbox" do |v|
    v.customize ["modifyvm", :id, "--memory", "2048", "--ioapic", "on", "--cpus", "4"]
  end

  config.vm.synced_folder ".", "/vagrant"

  # install docker
  config.vm.provision "shell", inline: <<-SCRIPT
    if [[ ! `which docker > /dev/null 2>&1` ]]; then
      # add docker's gpg key
      apt-key adv \
        --keyserver hkp://p80.pool.sks-keyservers.net:80 \
        --recv-keys 58118E89F3A912897C070ADBF76221572C52609D

      # add the source to our apt sources
      echo \
        "deb https://apt.dockerproject.org/repo ubuntu-trusty main \n" \
          > /etc/apt/sources.list.d/docker.list

      # update the package index
      apt-get -y update

      # ensure the old repo is purged
      apt-get -y purge lxc-docker

      # install docker
      apt-get -y install docker-engine

      # clean up packages that aren't needed
      apt-get -y autoremove

      # add the vagrant user to the docker group
      usermod -aG docker vagrant
    fi
  SCRIPT

  # start docker
  config.vm.provision "shell", inline: <<-SCRIPT
    if [[ ! `service docker status | grep "start/running"` ]]; then
      # start the docker daemon
      service docker start
    fi
  SCRIPT

  # wait for docker to be running
  config.vm.provision "shell", inline: <<-SCRIPT
    echo "Waiting for docker sock file"
    while [ ! -S /var/run/docker.sock ]; do
      sleep 1
    done
  SCRIPT

    # wait for docker to be running
  config.vm.provision "shell", inline: <<-SCRIPT
    apt-get install -y awscli
  SCRIPT

    # build the docker image
  config.vm.provision "shell", inline: <<-SCRIPT
    echo "Building docker image..."
    cd /vagrant
    docker build -t nanobox/build-openvpn-linux --no-cache=true -f linux/Dockerfile linux
    docker build -t nanobox/build-openvpn-mac --no-cache=true -f mac/Dockerfile mac
    docker build -t nanobox/build-openvpn-windows --no-cache=true -f windows/Dockerfile windows
  SCRIPT

end
