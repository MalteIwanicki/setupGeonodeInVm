# 0. What Is This?
We need to setup [Geonode](https://geonode.org/) with it's newest version to develop on.
Oracle's VirtualBox is needed.

# 1. Setup VM
In the VM window:
* Follow installation wizard in VM with an ubuntu 22.04 image
    1. Setup name (e.g. gn)
    2. Setup password (e.g. gn)
    3. Choose your keyboard layout
    4. Restart

In the terminal:
* update os with `sudo apt update && sudo apt upgrade`
* sudo apt install openssh-server -y

# 2. Open Ports
To be able to access the webpage outside the VM we need to open port 8080. To work headless on the VM we need to open port 3022.

VirtualBox > YOurVirtualMqachine > Settings > Network > Advanced > Port Forward > add new port forwarding rule

| Name | Protocol | Host IP | Guest IP | Guest Port |
| --- | --- | --- | --- | --- |
| ssh | TCP | 3022 | | 22 |
| 8080 | TCP | 8080 | | 80 |

# 3. Setup SSH
In Powershell on host machine:
ssh config
```shell
cd ~/.ssh
"
Host gn
 HostName 127.0.0.1
 Port 3022
 user gn
 IdentityFile ~/.ssh/vm_rsa
" | Out-File config -Append -Encoding utf8
```

setup rsa key
```shell
ssh-keygen
# Enter file in which to save the key (C:\Users\<user>/.ssh/id_rsa): 
vm_rsa


scp vm_rsa.pub gn:~/.ssh/authorized_keys
# Are you sure you want to continue connecting (yes/no/[fingerprint])?
yes
# geonode-vm-321@127.0.0.1's password:
geonode
```

connect via ssh
```shell
ssh gn
```
> Now we can work on the VM via ssh. Start VM in headless mode and connect in VSC-Remote_Explorer or in a terminal use <br>
 `ssh gn`

# 4. Setup Develop Environment
## Setup Virtual Environment
### Install Miniconda
```console
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm -rf ~/miniconda3/miniconda.sh
~/miniconda3/bin/conda init bash
~/miniconda3/bin/conda init zsh
```
### Create And Activate Venv
```console
conda create -n nfdi4bio
echo "conda activate nfdi4bio" >>~/.bashrc
conda activate nfdi4bio
```
## Install git
```
sudo apt-get install git
```
## Install docker
```
sudo snap install docker
```
## Download Repo
```
git clone https://github.com/Geonode-SEP-NFDI4Biodiversity/geonode ~/repositories/geonode
cd ~/repositories/geonode
```
## Download Other Files
Download files [here](https://github.com/Geonode-SEP-NFDI4Biodiversity/docker-compose-setup/find/main
) into `~/repositories/geonode`
## Switch Branch
```
git switch nfdi4bio_development
```
## Setup Docker
### Give Rights To User
```console
sudo groupadd docker
sudo usermod -aG docker $USER
su $USER
```
### Build
```console
docker build -t geonode_dev .
sudo service apache2 stop
docker compose up -d
```

# 5. Test
## Give Rights to users
attach geonode/postgis to shell (I used vsc)
```
psql -U postgres
\du
ALTER USER geonode Superuser;
ALTER USER geonode_data Superuser;
```

## Execute Tests
docker compose exec django manage.py test --failfast


## pycsw
### Upgrade 
`pip install git+https://github.com/geopython/pycsw.git`

### Downgrade
`pip install pycsw==2.6.1`
