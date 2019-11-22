

# Command

```shell
# 启动
进入Vagrantfile文件所在目录
vagrant up node1 [node2][..]
# ssh连接
vagrant ssh node1
```

## 快照





# Vagrantfile

```shell
Vagrant.configure("2") do |config|
  config.vm.box = "/Volumes/woody_1/vagrant/CentOS-7-x86_64-Vagrant-1910_01.VirtualBox.box"  
  config.vm.define :node1 do |node1|
      node1.vm.hostname = "node1"
      node1.vm.network "private_network", ip: "192.168.55.121"
      node1.vm.provider "virtualbox" do |v|
        v.name = "m1"
        v.memory = "2048"
        v.cpus = "2"
      end
  end
  config.vm.define :node2 do |node2|
      node2.vm.hostname = "node2"
      node2.vm.network :private_network, ip: "192.168.55.122"
      node2.vm.provider "virtualbox" do |v|
          v.name = "m2"
          v.memory = "2048"
          v.cpus = "2"
      end
  end
  config.vm.define :node3 do |node3|
      node3.vm.hostname = "node3"
      node3.vm.network "private_network", ip: "192.168.55.123"
      node3.vm.provider "virtualbox" do |v|
          v.name = "m3"
          v.memory = "1024"
          v.cpus = "1"
      end
  end
  config.vm.define :node4 do |node4|
      node4.vm.hostname = "node4"
      node4.vm.network "private_network", ip: "192.168.55.124"
      node4.vm.provider "virtualbox" do |v|
          v.name = "s1"
          v.memory = "1024"
          v.cpus = "1"
      end
  end
  config.vm.define :node5 do |node5|
      node5.vm.hostname = "node5"
      node5.vm.network "private_network", ip: "192.168.55.125"
      node5.vm.provider "virtualbox" do |v|
          v.name = "s2"
          v.memory = "1024"
          v.cpus = "1"
      end
  end
end
```

