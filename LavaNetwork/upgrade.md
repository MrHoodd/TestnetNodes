```bash
cd $HOME
rm -rf lava
git clone https://github.com/lavanet/lava.git
cd lava
git checkout v0.4.3
make build
sudo mv build/lavad /usr/local/bin/lavad
sudo systemctl restart lavad && sudo journalctl -u lavad -f -o cat
```
