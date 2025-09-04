Vagrant.configure("2") do |config|
  config.hostmanager.enabled = true 
  config.hostmanager.manage_host = true
  
### Master1 vm  ####
  config.vm.define "master1" do |master1|
    master1.vm.box = "ubuntu/jammy64"
    master1.vm.hostname = "master1"
    master1.vm.network "private_network", ip: "10.0.0.10"
    master1.vm.provider "virtualbox" do |vb|
     vb.memory = "2048"
   end
  end

### Master2 vm  ####
  config.vm.define "master2" do |master2|
    master2.vm.box = "ubuntu/jammy64"
    master2.vm.hostname = "master2"
    master2.vm.network "private_network", ip: "10.0.0.11"
    master2.vm.provider "virtualbox" do |vb|
     vb.memory = "2048"
     vb.cpus = 2
   end
  end  

### Master3 vm  ####
  config.vm.define "master3" do |master3|
    master3.vm.box = "ubuntu/jammy64"
    master3.vm.hostname = "master3"
    master3.vm.network "private_network", ip: "10.0.0.12"
    master3.vm.provider "virtualbox" do |vb|
     vb.memory = "2048"
     vb.cpus = 2
   end
  end  

### Worker1 vm  ####
  config.vm.define "worker1" do |worker1|
    worker1.vm.box = "ubuntu/jammy64"
    worker1.vm.hostname = "worker1"
    worker1.vm.network "private_network", ip: "10.0.0.13"
    worker1.vm.provider "virtualbox" do |vb|
     vb.memory = "2048"
     vb.cpus = 2
   end
  end
  
### Worker2 vm  ####
  config.vm.define "worker2" do |worker2|
    worker2.vm.box = "ubuntu/jammy64"
    worker2.vm.hostname = "worker2"
    worker2.vm.network "private_network", ip: "10.0.0.14"
    worker2.vm.provider "virtualbox" do |vb|
     vb.memory = "2048"
     vb.cpus = 2
   end
  end  

end
