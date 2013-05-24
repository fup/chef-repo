# -*- mode: ruby -*-
# vi: set ft=ruby :

# Install the chef client from a downloaded package
CHEF_SERVER_INSTALL = <<-EOF
#!/bin/sh
test -d /opt/chef-server || {
  echo "Installing chef-server via omnibus"
  apt-get install -y curl
  curl -L -s 'http://www.opscode.com/chef/download-server?p=ubuntu&pv=12.04&m=x86_64' > chef-server.dpkg
  dpkg -i chef-server.dpkg
  /opt/chef-server/bin/chef-server-ctl reconfigure >/dev/null
}
EOF

# Install the "workstation" in the project repository and configure knife to connect to the chef server
CHEF_CREATE_WORKSTATION = <<-EOF
#!/bin/sh
mkdir -p /vagrant/.chef
cp /etc/chef-server/admin.pem /vagrant/.chef/
cp /etc/chef-server/chef-validator.pem /vagrant/.chef/
cat <<EOK > /vagrant/.chef/knife.rb
cwd                     = File.dirname(__FILE__)
log_level               :info   # valid values - :debug :info :warn :error :fatal
log_location            STDOUT
node_name               ENV.fetch('KNIFE_NODE_NAME', 'admin')
client_key              ENV.fetch('KNIFE_CLIENT_KEY', File.join(cwd,'admin.pem'))
chef_server_url         ENV.fetch('KNIFE_CHEF_SERVER_URL', 'https://192.168.34.10')
validation_client_name  ENV.fetch('KNIFE_CHEF_VALIDATION_CLIENT_NAME', 'chef-validator')
validation_key          ENV.fetch('KNIFE_CHEF_VALIDATION_KEY', File.join(cwd,'chef-validator.pem'))
syntax_check_cache_path File.join(cwd,'syntax_check_cache')
cookbook_path           [File.join(cwd,'..','cookbooks'), File.join(cwd,'..','site-cookbooks')]
data_bag_path           File.join(cwd,'..','data_bags')
role_path               File.join(cwd,'..','roles')
EOK
EOF

Vagrant.configure('2') do |config|
  config.vm.define :chefserver do |node|
    node.vm.box_url = 'https://s3.amazonaws.com/vagrant_wd/wd-ubuntu-12.04.box'
    node.vm.box = 'wd-ubuntu-12.04'

    config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", 1024]
    end
    node.vm.box = 'wd-ubuntu-12.04'
    node.vm.network :private_network, ip: "192.168.34.10"
    node.vm.hostname = "chefserver.test.vm"
    node.vm.provision :shell, :inline => CHEF_SERVER_INSTALL
    node.vm.provision :shell, :inline => CHEF_CREATE_WORKSTATION
    node.vm.network :forwarded_port, guest: 80, host: 8080
    node.vm.network :forwarded_port, guest: 443, host: 8443
  end

  config.vm.define :chefclient do |node|
    node.vm.network :private_network, ip: "192.168.34.11"
    node.vm.box_url = 'https://s3.amazonaws.com/vagrant_wd/wd-ubuntu-12.04.box'
    node.vm.box = 'wd-ubuntu-12.04'
    node.vm.hostname = 'chefclient.test.vm'

    # configure the chef client node to run as a client connecting to the chef server
    config.vm.provision :chef_client do |chef|
      chef.chef_server_url = "https://192.168.34.10/"
      chef.validation_key_path = "./.chef/chef-validator.pem"
    end
  end

  config.vm.define :chefsolo do |node|
    node.vm.box_url = 'https://s3.amazonaws.com/vagrant_wd/wd-ubuntu-12.04.box'
    node.vm.box = 'wd-ubuntu-12.04'
    node.vm.hostname = 'chefsolo.test.vm'

    node.vm.provision :chef_solo do |chef|
       chef.add_role "base"
       chef.add_recipe "apache"
       chef.cookbooks_path = ["cookbooks"]
       chef.roles_path = "roles"
       chef.data_bags_path = "data_bags"
       chef.json = { }
    end
  end
end
