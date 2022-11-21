# Official Links
[Website](https://exorde.network/) [Twitter](https://twitter.com/ExordeLabs) [Discord](https://discord.gg/ExordeLabs)

# Explorer
[Explorer](https://explorer.exorde.network/) [Leaderboard](https://explorer.exorde.network/leaderboard)
# Install Node Guide
### Update Packages and Depencies
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install wget git jq build-essential nano curl unzip -y
```
### Install Docker
Update the apt package index and install packages to allow apt to use a repository over HTTPS:
```bash
sudo apt-get update &&
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
Add Docker’s official GPG key:
```bash
sudo mkdir -p /etc/apt/keyrings &&
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
Use the following command to set up the repository:
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
To install the latest version, run:
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
### Setting a variable with the address of your wallet
```bash
echo "export EXORDE_ADDRESS=<YOUR_METAMASK_WALLET_ADDRESS>" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
### Download the image and perform the installation
```bash
git clone https://github.com/exorde-labs/ExordeModuleCLI.git $HOME/ExordeModuleCLI \
cd $HOME/ExordeModuleCLI \
docker build -t exorde-cli . \
docker run -d -e PYTHONUNBUFFERED=1 --restart always --name exorde exorde-cli -m $EXORDE_WALLET -l 2
```
### Update
```bash
docker stop exorde
docker rm exorde
docker image prune -af
cd $HOME/ExordeModuleCLI
git pull
docker build -t exorde-cli .
docker run -d -e PYTHONUNBUFFERED=1 --restart always --name exorde exorde-cli -m $EXORDE_WALLET -l 2
```
### Deleting
```bash
docker stop exorde
docker rm exorde
docker image prune -af
rm -rf $HOME/ExordeModuleCLI
```
### Viewing logs
```bash
docker logs -f exorde
```
### Container restart
```bash
docker restart exorde
```
### Check your position in the leaderboard
```bash
curl -s https://raw.githubusercontent.com/exorde-labs/TestnetProtocol/main/Stats/leaderboard.json|grep $EXORDE_ADDRESS
```
### Stop all containers
```bash
docker stop $(docker ps -a -q)
```
### Delete all containers
```bash
docker rm $(docker ps -a -q)
```
