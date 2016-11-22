# my-vagrant
Here's some plugins or configs which I used on vagrant.

## Plugins
- [vagrant-hostmanager](#vagrant-hostmanager)
- [vagrant-share(system)](#vagrant-share)
- [vagrant-vbguest](#vagrant-vbguest)
- [vagrant-winnfsd](#vagrant-winnfsd)

### vagrant-hostmanager
<https://github.com/devopsgroup-io/vagrant-hostmanager>

配置域名本地解析，可以将自定义域名自动解析到 vagrant 主机。

> 注意: 至少有一个 `private_network` 网络连接，用于解析。

示例配置：
```rb
# -*- mode: ruby -*-
# vi: set ft=ruby :

domains = {
  frontend: 'www.dev.yiizh.com',
}

Vagrant.configure(2) do |config|

  config.vm.box = "yiizh/php-7.0"
  config.vm.hostname = "yiizh.dev"

  config.vm.provider "virtualbox" do |v|
    v.memory = 1024
  end

  config.vm.network 'private_network', type: "dhcp"

  # disable folder '/vagrant' (guest machine)
  config.vm.synced_folder ".", "/vagrant", type: "nfs"

  config.vm.provision :hostmanager
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = false
  config.hostmanager.aliases = domains.values

  config.hostmanager.ip_resolver = proc do |vm, resolving_vm|
    if hostname = (vm.ssh_info && vm.ssh_info[:host])
      `vagrant ssh -c "hostname -I"`.split()[1]
    end
  end

  config.vm.provision 'shell', path: './vagrant/provision/once-as-root.sh'
  config.vm.provision 'shell', path: './vagrant/provision/once-as-vagrant.sh', privileged: false
  config.vm.provision 'shell', path: './vagrant/provision/always-as-root.sh', run: 'always'
end
```

### vagrant-share
Vagrant 系统内置，用于挂载目录.

### vagrant-vbguest
https://github.com/dotless-de/vagrant-vbguest

自动安装与宿主机 virtualbox 相匹配的 vbguest.

### vagrant-winnfsd
https://github.com/winnfsd/vagrant-winnfsd

nfs 挂载插件。