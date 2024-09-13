Dependencies Installation

**Install dependencies for building from source**
```
sudo apt update
sudo apt install -y curl git jq lz4 build-essential
```

**Install Go**
```
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
Node Installation
```

**Download binary**
```
cd $HOME
curl -s https://snapshots-testnet.nodejumper.io/arkeonetwork-testnet/arkeod > arkeod
chmod +x arkeod
mkdir -p $HOME/go/bin/
mv arkeod $HOME/go/bin/
```

**Set node CLI configuration**
```
arkeod config chain-id arkeo
arkeod config keyring-backend test
arkeod config node tcp://localhost:26657
```

**Initialize the node**
```
arkeod init "Your Node Name" --chain-id arkeo
```

**Download genesis and addrbook files**
```
curl -L https://snapshots-testnet.nodejumper.io/arkeonetwork-testnet/genesis.json > $HOME/.arkeo/config/genesis.json
curl -L https://snapshots-testnet.nodejumper.io/arkeonetwork-testnet/addrbook.json > $HOME/.arkeo/config/addrbook.json
```

**Set seeds**
```
sed -i -e 's|^seeds *=.*|seeds = "20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:22856"|' $HOME/.arkeo/config/config.toml
```

**Set minimum gas price**
```
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.01uarkeo"|' $HOME/.arkeo/config/app.toml
```

**Set pruning**
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.arkeo/config/app.toml
```

**Download latest chain data snapshot**
```
curl "https://snapshots-testnet.nodejumper.io/arkeonetwork-testnet/arkeonetwork-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.arkeo"
```

**Create a service**
```
sudo tee /etc/systemd/system/arkeod.service > /dev/null << EOF
[Unit]
Description=Arkeo Network node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which arkeod) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable arkeod.service
```

**Start the service and check the logs**
```
sudo systemctl start arkeod.service
sudo journalctl -u arkeod.service -f --no-hostname -o cat
Secure Server Setup (Optional)
```

**generate ssh keys, if you don't have them already, DO IT ON YOUR LOCAL MACHINE**
```
ssh-keygen -t rsa
```

**save the output, we'll use it later on instead of YOUR_PUBLIC_SSH_KEY**
```
cat ~/.ssh/id_rsa.pub
# upgrade system packages
sudo apt update
sudo apt upgrade -y
```

**add new admin user**
```
sudo adduser admin --disabled-password -q
```

**upload public ssh key, replace YOUR_PUBLIC_SSH_KEY with the key above**
```
mkdir /home/admin/.ssh
echo "YOUR_PUBLIC_SSH_KEY" >> /home/admin/.ssh/authorized_keys
sudo chown admin: /home/admin/.ssh
sudo chown admin: /home/admin/.ssh/authorized_keys


echo "admin ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoer
```

**disable root login, disable password authentication, use ssh keys only**
```
sudo sed -i 's|^PermitRootLogin .*|PermitRootLogin no|' /etc/ssh/sshd_config
sudo sed -i 's|^ChallengeResponseAuthentication .*|ChallengeResponseAuthentication no|' /etc/ssh/sshd_config
sudo sed -i 's|^#PasswordAuthentication .*|PasswordAuthentication no|' /etc/ssh/sshd_config
sudo sed -i 's|^#PermitEmptyPasswords .*|PermitEmptyPasswords no|' /etc/ssh/sshd_config
sudo sed -i 's|^#PubkeyAuthentication .*|PubkeyAuthentication yes|' /etc/ssh/sshd_config


sudo systemctl restart sshd
```

**install fail2ban**
```
sudo apt install -y fail2ban
```

# install and configure firewall
sudo apt install -y ufw
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh
sudo ufw allow 9100
sudo ufw allow 26656

# make sure you expose ALL necessary ports, only after that enable firewall
sudo ufw enable

# make terminal colorful
sudo su - admin
source <(curl -s https://raw.githubusercontent.com/nodejumper-org/cosmos-scripts/master/utils/enable_colorful_bash.sh)

# update servername, if needed, replace YOUR_SERVERNAME with wanted server name
sudo hostnamectl set-hostname YOUR_SERVERNAME

# now you can logout (exit) and login again using ssh admin@YOUR_SERVER_IP
