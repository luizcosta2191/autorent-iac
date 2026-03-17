Vagrant.configure("2") do |config|
  # Define the Ubuntu Jammy (22.04) image
  config.vm.box = "ubuntu/jammy64"

  # Define a private network with a fixed IP to access the site
  config.vm.network "private_network", ip: "192.168.56.10"

  # Use the Ansible provisioner
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.become = true # Run as root (required to install packages)
  end
end
