# Ubuntu 20.04 Workstation Setup

## Introduction
I use my GPU-enabled workstation to run data science, ML, and DL workflows using a large number of open-source packages/projects. This includes, but is not limited to, RAPIDS, PyTorch, TensorRT, TensorFlow, multiple visualization packages, and much of the PyData ecosystem. I primarily develop using Docker containers, so the minimalist approach to setup here reflects that. These instructions are manual. If you prefer something more automated, my colleague [Paul](https://github.com/trxcllnt) has a [repo that automates setup of a new Xubuntu install](https://github.com/trxcllnt/ubuntu-setup).

## Setup Instructions
1. Install [Ubuntu 20.04](https://ubuntu.com/download/desktop)

2. Restart ([boot into recovery mode](https://wiki.ubuntu.com/RecoveryMode) if you’re having display issues)

3. Install GCC/G++


	```
	sudo apt install gcc
	```
	
4. Install [NVIDIA driver and CUDA toolkit](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=2004&target_type=debnetwork)

	```
	wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
	sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
	sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub
	sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/ /"
	sudo apt-get update
	sudo apt-get -y install cuda
	```
	
5. Install and configure SSH
	1. Install ZSH

		```
		sudo apt install openssh-server
		```

	2. Configure your client to connect to the workstation via SSH
		1. Instructions on how to [setup SSH keys and connect with keys](https://www.ssh.com/ssh/key/) [recommended]
		2. Insturctions on how to [connect via IP and password](https://linuxize.com/post/how-to-enable-ssh-on-ubuntu-20-04/)
	
7. Restart [optional, but gets you out of recovery mode]

8. Mount additional drives [optional]
	1. Edit `/etc/fstab` ([detailed instructions on how to find UUID and edit](https://askubuntu.com/questions/800695/hard-drive-wont-mount/800713#800713))
	2. Run `sudo mount -a` to reflect edited changes

9. Relink all folders with [symbolic links](https://linuxize.com/post/how-to-create-symbolic-links-in-linux-using-the-ln-command/) [optional]

	```
	# For every folder you want to symlink
	ln -s <path/to/source/folder> <path/to/target>
	```
	
10. Install git CLI tools

	```
	sudo apt install git
	```
	
11. Install curl 

	```
	sudo apt install curl
	```
	
12. Configure ZSH and set as default shell [optional, skip if you want to keep Bash]
	1. Install ZSH

		```
		sudo apt install zsh
		```

	2. Install [custom dotfiles](https://github.com/BartleyR/dotfiles) from GitHub repo

	3. Change default shell to ZSH

		```
		chsh -s $(which zsh)
		```

	4. Modify `path` to include link to CUDA binaries

		```
		# Add to ~/.aliases_functions.local
		path=('/usr/local/cuda/bin' $path)
		export PATH
		```

16. Install [Docker CE](https://docs.docker.com/engine/install/ubuntu/)

	```
	# Install dependencies
	sudo apt install \
	    apt-transport-https \
	    ca-certificates \
	    curl \
	    gnupg-agent \
	    software-properties-common
	
	# Add Docker GPG key
	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
	
	# Setup Docker stable repo
	sudo add-apt-repository \
	   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
	   $(lsb_release -cs) \
	   stable"
	
	# Install Docker engine
	sudo apt update
	sudo apt install docker-ce docker-ce-cli containerd.io
	```
	
17. Setup Docker to manage it as a non-root user
	
	```
	# Add the docker group (might already exist)
	sudo groupadd docker
	
	# Add your account to the docker group
	sudo usermod -aG docker $USER
	
	# Need to log out/in to reevaluate group membership
	```
	
18. Setup [NVIDIA container toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker)

	```
	# Setup stable repo and GPG key
	distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
	   && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
	   && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
	
	# Install nvidia-docker2 and dependencies
	sudo apt update
	sudo apt install -y nvidia-docker2
	
	# Restart Docker daemon
	sudo systemctl restart docker
	
	# Test by running a base CUDA container
	docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
	```
	
19. Verify NVLink is functioning [optional]

	```
	# Copy CUDA samples to home directory
	cuda-install-samples-11.1.sh ~/
	
	# Get in the right directory
	cd ~/NVIDIA_CUDA-11.1_Samples/0_Simple/simpleP2P/
	
	# Compile the sample
	make
	
	# Run the test
	./simpleP2P
	
	# Can also run the bandwidth latency test
	cd ~/NVIDIA_CUDA-11.1_Samples/1_Utilities/p2pBandwidthLatencyTest
	make
	./p2pBandwidthLatencyTest
	```

20. Install [AWS CLI tools](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html#cliv2-linux-install)

	```
	# Download AWS CLI file
	curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
	
	# Extract file
	unzip awscliv2.zip
	
	# Install AWS CLI
	sudo ./aws/install
	```
	
## Optional Next Steps
Sometimes (rarely these days) I might need multiple CUDA toolkit versions on the bare metal OS. For this latest install, I'm skipping this and going to try getting by with CUDA 11.x on the host OS while managing other CUDA toolkits via containers (if necessary). If you need multiple CUDA toolkit versions on your install, [Paul's script on how to configure this](https://github.com/trxcllnt/ubuntu-setup/blob/master/scripts/01-cuda.sh) is very useful.
	
## Acknowledgments
I relied heavily on [Paul's Xubuntu bootstrap scripts](https://github.com/trxcllnt/ubuntu-setup) to help make this simple walkthrough.