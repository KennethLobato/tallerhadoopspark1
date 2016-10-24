Vagrant.configure(2) do |config|
  #Define two virtual machines, ubuntu1 and ubuntu2
  config.vm.define "ubuntu1" do |ubuntu1|
    #Select the image that we want
    ubuntu1.vm.box = "ubuntu/trusty64"
    #Setup a private network that allows both machines comms.
    ubuntu1.vm.network "private_network", ip:"10.0.0.10"
    ubuntu1.vm.hostname = "ubuntu1"
    ubuntu1.vm.provision "shell", inline: <<-SHELL
      echo "Local Configuration"
      if [ ! -f /vagrant/shared/ubuntu1 ]; then
        ssh-keygen -t rsa -P "" -f /vagrant/shared/ubuntu1
      fi
      cp /vagrant/shared/ubuntu1 /home/vagrant/.ssh/id_rsa
      chown vagrant /home/vagrant/.ssh/id_rsa
      chmod 600 /home/vagrant/.ssh/id_rsa
      cp /vagrant/shared/ubuntu1.pub /home/vagrant/.ssh/id_rsa.pub
      cat /vagrant/shared/ubuntu1.pub >> /home/vagrant/.ssh/authorized_keys
    SHELL
  end

  config.vm.define "ubuntu2" do |ubuntu2|
    ubuntu2.vm.box = "ubuntu/trusty64"
    ubuntu2.vm.network "private_network", ip:"10.0.0.11"
    ubuntu2.vm.hostname = "ubuntu2"
    ubuntu2.vm.provision "shell", inline: <<-SHELL
      echo "Local Configuration"
      cp /vagrant/shared/ubuntu1 /home/vagrant/.ssh/id_rsa
      chown vagrant /home/vagrant/.ssh/id_rsa
      chmod 600 /home/vagrant/.ssh/id_rsa
      cp /vagrant/shared/ubuntu1.pub /home/vagrant/.ssh/id_rsa.pub
      cat /vagrant/shared/ubuntu1.pub >> /home/vagrant/.ssh/authorized_keys
    SHELL
  end

  config.vm.provider "virtualbox" do |vb|
    vb.memory = 4096
    vb.cpus = 2
  end

  config.vm.provision "shell", inline: <<-SHELL
    echo "Common Configuration"
    #Add Java 8 repository
    sudo apt-add-repository -y ppa:webupd8team/java

    #Update machine
    sudo apt-get update

    #Install outdated software
    sudo apt-get -y upgrade

    #Automatically Accept Java8 License
    sudo echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | sudo /usr/bin/debconf-set-selections

    #Install Java8
    sudo apt-get install -y oracle-java8-installer


    #Configure hostname resolution
    sudo echo "10.0.0.10 ubuntu1.tallerhadoop1.org ubuntu1" >> /etc/hosts
    sudo echo "10.0.0.11 ubuntu2.tallerhadoop1.org ubuntu2" >> /etc/hosts
    cat /etc/hosts | grep -v ^127.0.1.1 | grep -v ^f | grep -v ^: | grep -v ^# > hosts
    sudo cp hosts /etc/hosts
    rm hosts

    #Download Hadoop 2.7.3 Binary
    wget http://ftp.cixug.es/apache/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz -P /home/vagrant/
    echo "Donwloaded Hadoop 2.7.3 binaries"
    sudo tar -zxvf /home/vagrant/hadoop-2.7.3.tar.gz -C /usr/local/
    sudo ln -sf /usr/local/hadoop-2.7.3/ /usr/local/hadoop
    sudo chown -R vagrant /usr/local/hadoop/
    sudo tar -zxvf /vagrant/shared/hadoop-files.tar.gz -C /usr/local/hadoop/etc/
    sudo cp /usr/local/hadoop/etc/hadoop-files/* /usr/local/hadoop/etc/hadoop/
    sudo rm -rf /usr/local/hadoop/etc/hadoop-files

    #Setup bashrc
    echo "export JAVA_HOME=/usr/lib/jvm/java-8-oracle/" >> /home/vagrant/.bashrc
    echo "export HADOOP_HOME=/usr/local/hadoop" >> /home/vagrant/.bashrc
    echo "export PATH=$PATH:/usr/local/hadoop/bin" >> /home/vagrant/.bashrc
  SHELL
end
