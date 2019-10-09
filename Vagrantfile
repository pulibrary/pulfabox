# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # https://docs.vagrantup.com.
  # NB: you may need to upgrade or repair the virtualbox guest plugin
  # (see
  # https://www.serverlab.ca/tutorials/virtualization/how-to-auto-upgrade-virtualbox-guest-additions-with-vagrant/)

  config.vm.box = "ubuntu/xenial64"
  config.vm.network "private_network", type: "dhcp"
  config.vm.network "forwarded_port", guest: 80, host: 4567
  config.vm.network "forwarded_port", guest: 8080, host: 8586
  config.vm.network "forwarded_port", guest: 8182, host: 8587
  config.vm.network "forwarded_port", guest: 8183, host: 8588
  config.vm.network "forwarded_port", guest: 8983, host: 8984
  config.vm.provider "virtualbox" do |vb|
    vb.memory = 8192
    vb.cpus = 2
  end

  #
  # IMPORTANT !!!!
  #
  # You MUST set the synced folders below to link to folders on YOUR local (host) machine.
  # /data should be linked to the folder containing the SVN-managed data files;
  # /pulfa should be linked to the folder containing the pulfa code.
  
  config.vm.synced_folder "/Users/cwulfman/repos/github/pulibrary/data/trunk", "/data"
  config.vm.synced_folder "/Users/cwulfman/pulfa", "/pulfa"

  config.vm.provision "shell", inline: <<-SHELL
   apt-get update
   apt-get install -y apache2
   sudo a2enmod proxy proxy_http proxy_ajp rewrite
   sudo curl -o /etc/apache2/sites-available/000-default.conf https://raw.githubusercontent.com/pulibrary/pulfa-extras/master/vhost
   sudo service apache2 restart
   apt-get install -y default-jre
   apt-get install -y default-jdk
   sudo mkdir -p /var/lib/existdb /var/log/existdb /var/run/existdb
   cd /usr/local/lib
   sudo curl -o exist.tgz https://raw.githubusercontent.com/pulibrary/pulfa-extras/master/exist-1.4.3-configured.tgz
   sudo tar xzf exist.tgz
   sudo rm exist.tgz
   sudo groupadd existdb
   sudo useradd -g existdb existdb
   sudo chown -R existdb:existdb /usr/local/lib/exist/webapp /var/lib/existdb /var/log/existdb /var/run/existdb
   sudo mkdir /opt/apache
   cd /opt/apache
   sudo curl -o solr.tgz https://raw.githubusercontent.com/pulibrary/pulfa-extras/master/solr.tgz
   sudo tar xzf solr.tgz
   sudo rm solr.tgz
   sudo curl -o /etc/default/jetty https://raw.githubusercontent.com/pulibrary/pulfa-extras/master/jettyconfig
   sudo useradd -d /opt/apache/solr -s /sbin/false solr
   sudo chown solr:solr -R /opt/apache/solr
   sudo mkdir /var/log/solr
   sudo chown solr:solr /var/log/solr
   sudo ln -s /opt/apache/solr/jetty.sh /etc/init.d/solr
   sudo update-rc.d solr defaults
   sudo -u existdb /usr/local/lib/exist/bin/startup.sh &
   sudo service solr start

   apt-get install -y git-core subversion libxml2-utils cifs-utils ant
   sudo adduser pulfa --gecos "First Last,RoomNumber,WorkPhone,HomePhone" --disabled-password
   echo "pulfa:Du1135" | sudo chpasswd
   sudo ln -s /pulfa /home/pulfa/app
   sudo ln -s /pulfa/webapp /usr/local/lib/exist/webapp/pulfa

# very ugly work-around for the base-uri issue in the VM environment
  sudo sed -i -e 's|declare variable $local:base-uri as xs:string := local:resolve-base-uri();|declare variable $local:base-uri as xs:string := "http://localhost:4567";|' /usr/local/lib/exist/webapp/pulfa/controller.xql

   sudo mkdir /var/log/pulfa
   sudo chown pulfa:pulfa /var/log/pulfa

   sudo cp /pulfa/bin/conf.sh.tmpl /pulfa/bin/conf.sh
   sudo sed -i -e 's|PULFA_HOST=\"|PULFA_HOST=\"http://localhost/pulfa|' /pulfa/bin/conf.sh
   sudo sed -i -e 's/PULFA_DB_USER=\"/PULFA_DB_USER=\"pulfa/' /pulfa/bin/conf.sh
   sudo sed -i -e 's|/home/pulfa/data/|/data/|g' /pulfa/bin/conf.sh

   sudo cp /pulfa/build.xml.tmpl /pulfa/build.xml
   sudo sed -i 's/__EXIST_USER__/pulfa/' /pulfa/build.xml
   sudo sed -i 's/__EXIST_PW__/Du1135/' /pulfa/build.xml
   cd /home/pulfa/app; ant initializedb

   sudo apt-get upgrade python3
   apt-get install -y python3-venv
   apt-get install -y python3-pip
   apt-get install -y unzip

   curl -L -o /tmp/pulfamanager.zip https://github.com/pulibrary/pulfamanager/archive/master.zip
   unzip -d /usr/local/bin /tmp/pulfamanager.zip
   mv /usr/local/bin/pulfamanager-master /usr/local/bin/pulfamanager
   


   sudo cp /usr/local/bin/pulfamanager/conf.yaml.tmpl /usr/local/bin/pulfamanager/conf.yaml
   sudo sed -i 's|host: https://findingaids.princeton.edu|host: http://localhost:8080/exist/pulfa|' /usr/local/bin/pulfamanager/conf.yaml
   sudo sed -i 's|/home/pulfa/data/|/data/|g' /usr/local/bin/pulfamanager/conf.yaml
   sudo sed -i 's|pulfa_db_passwd:|pulfa_db_passwd: Du1135|' /usr/local/bin/pulfamanager/conf.yaml
   sudo sed -i 's|http://findingaids.princeton.edu:8983/solr|http://localhost:8983/solr|' /usr/local/bin/pulfamanager/conf.yaml
   sudo sed -i 's|saxon: \/home\/pulfa\/lib\/saxon9he.jar|saxon: \/pulfa\/lib\/saxon9he.jar|' /usr/local/bin/pulfamanager/conf.yaml

   

   cd /usr/local/bin/pulfamanager
   chmod a+x ./load_record.py
   sudo ln -s /usr/local/bin/pulfamanager/load_record.py /usr/local/bin/load_record
   python3 -m venv venv
   source venv/bin/activate
   pip install --upgrade pip
   pip install -r requirements.txt --no-cache-dir
   
    SHELL
end
