
Vagrant.configure("2") do |config|
  # L'OS
  config.vm.box = "debian/bookworm64"
  config.vm.hostname = "nas-srv"
  
  # IP
  config.vm.network "private_network", ip: "192.168.56.10"

  # Configuration VirtualBox
  config.vm.provider "virtualbox" do |vb|
    vb.name = "NAS_Debian"
    vb.memory = "2048"
    vb.cpus = 2
    # sans interface graphique
    vb.gui = false
  end

  # provisionnement  disques données
  (1..3).each do |i|
    config.vm.disk :disk, size: "10GB", name: "nas_data_#{i}"
  end
end