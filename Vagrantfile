# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # Use Ubuntu 18.04 LTS
  config.vm.box = "ubuntu/bionic64"

  # Expose Klipper source tree to VM
  mount_point = "/klipper_src"
  config.vm.synced_folder ".", mount_point

  config.vm.provider "virtualbox" do |vb|
    vb.cpus = 2
    vb.memory = 2048

    # Use linked clones for speed and to save space
    vb.linked_clone = true if Gem::Version.new(Vagrant::VERSION) >= Gem::Version.new("1.8.0")

    # Keep VM time synced to avoid time skew errors during build
    {
      '--timesync-interval' => '30000',  # check time every 10 seconds
      '--timesync-min-adjust' => '100',  # only adjust if at least 100ms out
      '--timesync-set-on-restore' => '1',  # set time on restore
      '--timesync-set-threshold' => '1000',  # adjust no more than 1s at a time
    }.each do |setting,value|
      vb.customize [
        'guestproperty', 'set', :id,
        "/VirtualBox/GuestAdd/VBoxService/#{setting}", value
      ]
    end
  end

  config.vm.provision "shell", inline: <<-SHELL
    export DEBIAN_FRONTEND=noninteractive

    # CI build environment doesn't confirm package installation
    echo 'APT::Get::Assume-Yes "true";' > /etc/apt/apt.conf.d/99assume-yes.conf

    # Start with an up-to-date VM
    apt-get update
    apt-get upgrade -y

    # Run CI build installation to install dependencies
    cd #{mount_point}
    ./scripts/ci-install.sh
  SHELL
end
