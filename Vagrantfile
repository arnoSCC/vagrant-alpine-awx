# -*- mode: ruby -*-
# vi: set ft=ruby :

VM_MEMORY=8172
VM_CORES=4

Vagrant.configure("2") do |config|
  config.vm.box = "generic/alpine38"

  config.vm.provider :vmware_desktop do |v, override|
    v.vmx['memsize'] = VM_MEMORY
    v.vmx['numvcpus'] = VM_CORES
  end

  config.vm.provider :virtualbox do |v, override|
    v.memory = VM_MEMORY
    v.cpus = VM_CORES
  end

  config.vm.provision "file", source: "./wheel/.", destination: "$HOME/"
  
  config.vm.provision "shell", inline: <<-SHELL
echo "vagrant.$(ip a show dev eth0 | grep -m1 inet | awk '{print $2}' | cut -d'/' -f1).xip.io" > /etc/hostname
hostname -F /etc/hostname
apk add python3 docker shadow curl
ln -s /usr/bin/python3 /usr/bin/python
rc-update add docker default
service docker start
usermod -aG docker vagrant
mv /home/vagrant/wheel /root/wheel && chown -R root:root /root/wheel
cd /root/wheel && python pip-19.0.3-py2.py3-none-any.whl/pip install * && cd /root/
curl -sSL https://github.com/ansible/awx/archive/3.0.1.tar.gz | tar xz
cd awx-3.0.1/installer && ansible-playbook -i inventory install.yml
echo "\n\nStart waiting for OKREADY in awx_task container..."
ID=$(docker ps --no-trunc -f name=^/awx_task$ -q)
while [ $(grep 'OKREADY' /var/lib/docker/containers/$ID/$ID-json.log | wc -l) -eq 0 ]; do sleep 5; done
echo "\nAWX is ready and accessible at http://$(ip a show dev eth0 | grep -m1 inet | awk '{print $2}' | cut -d'/' -f1)"
  SHELL
  

end
