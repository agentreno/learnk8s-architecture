BOX_IMAGE = "bento/ubuntu-18.04"
KUBEADM_VERSION = "1.13.5-00"

Vagrant.configure("2") do |config|
  config.vm.provider :virtualbox do |v|
    v.memory = 1024
    v.cpus = 1
  end

  config.vm.provision :shell, privileged: true, inline: $install_common_tools

  config.vm.define :master do |master|
    master.vm.box = BOX_IMAGE
    master.vm.hostname = "master"
    master.vm.network :private_network, ip: "192.168.0.10"
    master.vm.provider :virtualbox do |vb|
      vb.cpus = 2
    end
  end

  %w{worker1 worker2}.each_with_index do |name, i|
    config.vm.define name do |worker|
      worker.vm.box = BOX_IMAGE
      worker.vm.hostname = name
      worker.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "512"]
      end
      worker.vm.network :private_network, ip: "192.168.0.#{i + 11}"
    end
  end

  config.vm.provision "shell", inline: $install_multicast
end


$install_common_tools = <<-SCRIPT
# disable swap
swapoff -a
sed -i '/swap/d' /etc/fstab

# Install kubeadm, kubectl and kubelet
export DEBIAN_FRONTEND=noninteractive
apt-get -qq install ebtables ethtool
apt-get -qq update
apt-get -qq install -y docker.io apt-transport-https curl
apt-key adv --fetch-keys https://packages.cloud.google.com/apt/doc/apt-key.gpg
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get -qq update
apt-get -qq install -y \
  kubelet=#{KUBEADM_VERSION} \
  kubeadm=#{KUBEADM_VERSION} \
  kubectl=#{KUBEADM_VERSION}
SCRIPT

$install_multicast = <<-SHELL
apt-get -qq install -y avahi-daemon libnss-mdns
SHELL
