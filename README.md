# What Is This?
We need to setup [Geonode](https://geonode.org/) with it's newest version to develop on.
Oracle's VirtualBox is needed.<sub><sup>This is not 100% complete and could miss some steps.</sup></sub>

* [Setup](#setup)
* [Test](#test)
* [Debug](#debug)

<a name="setup"></a>
# Setup
## 1. Setup VM
In the VM window:
* Follow installation wizard in VM with an ubuntu 22.04 image
    1. Setup name (e.g. gn)
    2. Setup password (e.g. gn)
    3. Choose your keyboard layout
    4. Restart

In the terminal:
* update os with `sudo apt update && sudo apt upgrade`
* sudo apt install openssh-server -y

## 2. Open Ports
To be able to access the webpage outside the VM we need to open port 8080. To work headless on the VM we need to open port 3022.

VirtualBox > YOurVirtualMqachine > Settings > Network > Advanced > Port Forward > add new port forwarding rule

| Name | Protocol | Host IP | Guest IP | Guest Port |
| --- | --- | --- | --- | --- |
| ssh | TCP | 3022 | | 22 |
| 8080 | TCP | 8080 | | 80 |

## 3. Setup SSH
In Powershell on host machine:
ssh config
```console
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
```console
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
```console
ssh gn
```
> Now we can work on the VM via ssh. Start VM in headless mode and connect in VSC-Remote_Explorer or in a terminal use <br>
 `ssh gn`

## 4. Setup Develop Environment
### Setup Virtual Environment
#### Install Miniconda
```console
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm -rf ~/miniconda3/miniconda.sh
~/miniconda3/bin/conda init bash
~/miniconda3/bin/conda init zsh
```
#### Create And Activate Venv
```console
conda create -n nfdi4bio
echo "conda activate nfdi4bio" >>~/.bashrc
conda activate nfdi4bio
```
### Install git
```console
sudo apt-get install git
```
### Install docker
```console
sudo snap install docker
```
## 5. Setup Files
### Clone Repos
```console
git clone https://github.com/Geonode-SEP-NFDI4Biodiversity/geonode ~/repositories/geonode
git clone git+https://github.com/Geonode-SEP-NFDI4Biodiversity/docker-compose-setup.git ~/repositories/docker-compose-setup
```
### Copy Files And Make Executable
```console
cd ~/repositories/geonode
cp ../docker-compose-setup/.env_nfdi4bio .
cp ../docker-compose-setup/docker-compose-nfdi4bio.yml .
cp ../docker-compose-setup/docker-compose-nfdi4bio.sh .
sudo chmod +x docker-compose-nfdi4bio.sh
```
#### Stop Git Tracking Files
```console
printf "\n.env_nfdi4bio" >> .git/info/exclude
printf "\ndocker-compose-nfdi4bio.yml" >> .git/info/exclude
printf "\ndocker-compose-nfdi4bio.sh" >> .git/info/exclude
```
### Switch Branch
```console
git switch nfdi4bio_development
```

## 6. Setup Docker
### Give Rights To User
```console
sudo groupadd docker
sudo usermod -aG docker $USER
su $USER
```
### Build
```console
dotenv --dotenv .env_nfdi4bio docker build -t geonode_dev .
```
> When making changes to python files, you have to reload WSGI server:
> ```console
> ./docker-compose-nfdi4bio.sh exec django touch /usr/src/geonode/geonode/wsgi.py
> ```
### Start
```console
sudo service apache2 stop
dotenv --dotenv .env_nfdi4bio docker compose -f docker-compose-nfdi4bio.yml "$@"
```
### Give DB Rights To Users
attach geonode/postgis to shell (I used vsc)
```
psql -U postgres
\du
ALTER USER geonode Superuser;
ALTER USER geonode_data Superuser;
```

<a name="test"></a>
# Test
## Execute Tests
```console
./docker-compose-nfdi4bio.sh exec django manage.py test --failfast
```

<a name="debug"></a>
# Debug
1. Attached the container to VSC
2. Installed the python debugger extension on new VSC window, then reload window
3. Created the launch json with:
```console
{
            "name": "Test",
            "type": "python",
            "request": "launch",
            "program": "${workspaceFolder}/manage.py",
            "console": "integratedTerminal",
            "justMyCode": false,
            "args": [
                "test", "geonode.catalogue", "--failfast"
            ]
}
```
4. Set Breakpoints
5. Run & Debug Test
