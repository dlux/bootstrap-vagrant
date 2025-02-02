# -*- mode: ruby -*-
# vi: set ft=ruby :
##############################################################################
# Copyright (c)
# All rights reserved. This program and the accompanying materials
# are made available under the terms of the Apache License, Version 2.0
# which accompanies this distribution, and is available at
# http://www.apache.org/licenses/LICENSE-2.0
##############################################################################

$no_proxy = ENV['NO_PROXY'] || ENV['no_proxy'] || "127.0.0.1,localhost"
# NOTE: This range is based on vagrant-libvirt network definition CIDR 192.168.121.0/24
(1..254).each do |i|
  $no_proxy += ",192.168.121.#{i}"
end
$no_proxy += ",10.0.2.15"

Vagrant.configure("2") do |config|
  config.vm.provider :libvirt
  config.vm.provider :virtualbox

  provider = (ENV['PROVIDER'] || :virtualbox).to_sym
  config.vm.define "ubuntu_#{provider}" do |ubuntu|
    ubuntu.vm.box = 'elastic/ubuntu-16.04-x86_64'
    ubuntu.vm.box_version = '20180210.0.0'
  end
  config.vm.define "centos_#{provider}" do |centos|
    centos.vm.box = 'centos/7'
    centos.vm.box_version = '1905.01'
  end
  config.vm.define "opensuse_#{provider}" do |opensuse|
    opensuse.vm.box = 'opensuse/openSUSE-Tumbleweed-Vagrant.x86_64'
    opensuse.vm.box_version = '1.0.20190918'
  end
  # Upgrade Kernel version
  config.vm.provision 'shell', privileged: false, inline: <<-SHELL
    source /etc/os-release || source /usr/lib/os-release
    case ${ID,,} in
        rhel|centos|fedora)
        PKG_MANAGER=$(command -v dnf || command -v yum)
        INSTALLER_CMD="sudo -H -E ${PKG_MANAGER} -q -y install"
        if ! sudo "$PKG_MANAGER" repolist | grep "epel/"; then
            $INSTALLER_CMD epel-release
        fi
        sudo "$PKG_MANAGER" updateinfo
        $INSTALLER_CMD kernel
        sudo grub2-set-default 0
        sudo grub2-mkconfig -o "$(sudo readlink -f /etc/grub2.cfg)"
        ;;
    esac
  SHELL
  config.vm.provision :reload
  config.vm.provision 'shell', privileged: false do |sh|
    sh.env = {
      'DEBUG': "true"
    }
    sh.inline = <<-SHELL
      set -o xtrace
      cd /vagrant/
      PROVIDER=#{provider} ./setup.sh
    SHELL
  end
  config.vm.provision :reload
  config.vm.provision 'shell', privileged: false do |sh|
    sh.inline = <<-SHELL
      cd /vagrant/
      source /etc/os-release || source /usr/lib/os-release
      ./validate.sh | tee validate_${ID,,}_#{provider}.log
    SHELL
  end

  [:virtualbox, :libvirt].each do |provider|
  config.vm.provider provider do |p, override|
      p.cpus = 4
      p.memory = 8192
    end
  end

  config.vm.provider :libvirt do |v, override|
    v.cpu_mode = 'host-passthrough'
    v.nested = true
    v.random_hostname = true
    v.management_network_address = "192.168.121.0/24"
  end

  if ENV['http_proxy'] != nil and ENV['https_proxy'] != nil
    if Vagrant.has_plugin?('vagrant-proxyconf')
      config.proxy.http     = ENV['http_proxy'] || ENV['HTTP_PROXY'] || ""
      config.proxy.https    = ENV['https_proxy'] || ENV['HTTPS_PROXY'] || ""
      config.proxy.no_proxy = $no_proxy
    end
  end
end
