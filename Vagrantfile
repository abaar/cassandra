Vagrant.configure("2") do |config|
  config.vm.define "cassandra2" do |cassandra2|
    # Box Settings
    cassandra2.vm.box = "bento/ubuntu-16.04"
    cassandra2.vm.hostname = "cassandra2"

    # Provider Settings
    cassandra2.vm.provider "virtualbox" do |vb|
      # Display the VirtualBox GUI when booting the machine
      vb.gui = false
      vb.name = "cassandra2"
      # Customize the amount of memory on the VM:
      vb.memory = "2048"
    end
    cassandra2.vm.network "private_network", ip: "192.168.2.3"
    cassandra2.vm.provision "shell", path: "provision/bootstrap.sh", privileged: false
    # cassandra2.vm.provision "shell", path: "provision/bootstrap-cassandra2.sh", privileged: false
  end

  config.vm.define "cassandra1" do |cassandra1|
    # Box Settings
    cassandra1.vm.box = "bento/ubuntu-16.04"
    cassandra1.vm.hostname = "cassandra1"

    # Provider Settings
    cassandra1.vm.provider "virtualbox" do |vb|
      # Display the VirtualBox GUI when booting the machine
      vb.gui = false
      vb.name = "cassandra1"
      # Customize the amount of memory on the VM:
      vb.memory = "2048"
    end
    cassandra1.vm.network "private_network", ip: "192.168.2.2"
    cassandra1.vm.provision "shell", path: "provision/bootstrap.sh", privileged: false
    # cassandra1.vm.provision "shell", path: "provision/bootstrap-cassandra1.sh", privileged: false
  end
end
