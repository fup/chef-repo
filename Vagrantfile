Vagrant.configure('2') do |config|
  config.vm.define "chef_client" do |node|
    node.vm.box_url = 'https://s3.amazonaws.com/vagrant_wd/wd-ubuntu-12.04.box'
    node.vm.hostname = 'cheftraining.test.vm'
    node.vm.box = 'wd-ubuntu-12.04'

    node.vm.provision :chef_solo do |chef|
       chef.add_recipe "apache"
       chef.cookbooks_path = ["cookbooks"]
       chef.roles_path = "roles"
       chef.data_bags_path = "data_bags"
       chef.json = { }
    end
  end
end
