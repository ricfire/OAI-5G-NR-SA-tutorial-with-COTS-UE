# OAI-5G-NR-SA-tutorial-with-COTS-UE
Tutorial completo para implantação e configuração de um testbed utilizando o OpenAirInterface (OAI), incluindo instalação, configuração do core e RAN, execução de experimentos e troubleshooting.

## 1. Cenário

Neste tutorial, descrevemos como configurar e executar uma instalação 5G de ponta a ponta com OAI CN5G, OAI gNB e UE COTS.

Hardware e Software utilizados:


* Workstation Dell-5860 com Xeon w3-2425, RTX 4000 ADA, 64GB
de RAM e 1TB de SSD Nvme para OAI CN5G e OAI gNB
* SDR USRP-X310
* Sistema operacional: Ubuntu 24.04 LTS


Quectel RM500Q

Módulo, adaptador M.2 para USB, antenas e cartão SIM
A versão do firmware do Quectel DEVE ser igual ou superior a RM500QGLABR11A06M4G

## 2. OAI CN5G

### 2.1 Pré-requisitos do OAI CN5G
```bash
sudo apt install -y git net-tools putty

# https://docs.docker.com/engine/install/ubuntu/
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add your username to the docker group
sudo usermod -a -G docker $(whoami)
reboot
```
### 2.2 Arquivos de configuração do OAI CN5G

Baixe e copie os arquivos de configuração:
```bash
wget -O ~/oai-cn5g.zip https://gitlab.eurecom.fr/oai/openairinterface5g/-/archive/develop/openairinterface5g-develop.zip?path=doc/tutorial_resources/oai-cn5g
unzip ~/oai-cn5g.zip
mv ~/openairinterface5g-develop-doc-tutorial_resources-oai-cn5g/doc/tutorial_resources/oai-cn5g ~/oai-cn5g
rm -r ~/openairinterface5g-develop-doc-tutorial_resources-oai-cn5g ~/oai-cn5g.zip
```
### 2.3 Baixar imagens Docker do OAI CN5G
```bash
cd ~/oai-cn5g
docker compose pull
```
## 3. OAI gNB
### 3.1 Pré-requisitos do OAI gNB
Compilar o UHD a partir do código-fonte
```bash
# https://files.ettus.com/manual/page_build_guide.html
sudo apt install -y autoconf automake build-essential ccache cmake cpufrequtils doxygen ethtool g++ git inetutils-tools libboost-all-dev libncurses-dev libusb-1.0-0 libusb-1.0-0-dev libusb-dev python3-dev python3-mako python3-numpy python3-requests python3-scipy python3-setuptools python3-ruamel.yaml

git clone https://github.com/EttusResearch/uhd.git ~/uhd
cd ~/uhd
git checkout v4.8.0.0
cd host
mkdir build
cd build
cmake ../
make -j $(nproc)
make test # This step is optional
sudo make install
sudo ldconfig
sudo uhd_images_downloader
```
### 3.2 Construir a OAI gNB
```bash
# Get openairinterface5g source code
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git ~/openairinterface5g
cd ~/openairinterface5g
git checkout develop

# Install OAI dependencies
cd ~/openairinterface5g/cmake_targets
./build_oai -I

# Build OAI gNB
cd ~/openairinterface5g/cmake_targets
./build_oai -w USRP --ninja --gNB -C
```
## 4. Executar o OAI CN5G e o OAI gNB
### 4.1 Iniciar o OAI CN5G
```bash
cd ~/oai-cn5g
docker compose up -d
```
### 4.2 Iniciar o OAI gNB
```bash
cd ~/oai-cn5g
docker compose down
```

