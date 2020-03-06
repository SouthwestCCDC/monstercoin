monsterCoin
================================

Copyright (c) 2009-2013 Bitcoin Developers
Copyright (c) 2011-2013 Litecoin Developers

What is monsterCoin?
----------------
Monsters need to spend money just like everybody else.

Ubuntu 18.04 Installation
------------
Below are commands necessary to install MonsterCoin on ubuntu 18.04. Note that some of the commands are interactive. As such you will need to execute line by line.
```
sudo -s

cd ~/
git clone https://github.com/SouthwestCCDC/monstercoin.git
cd ~/monstercoin/src

# install dependencies
apt-get update
apt-get install libdb4.8++ libdb++-dev libboost-dev miniupnpc libboost-all-dev libminiupnpc-dev build-essential libssl1.0-dev

# create user for service to run as
adduser monstercoin

# make config dir
mkdir /home/monstercoin/.monstercoin

# handle bug
mkdir obj

# In the real world, a smart person would change this password per deployment
# Since we're lazy, we're just going to use this everywhere
cat << EOF > /home/monstercoin/.monstercoin/monstercoin.conf
rpcuser=monstercoinrpc
rpcpassword=C2xr7dcABXAmhvidci7Q24LNR1VzRVyRpWWLtEqk9k8P
addnode=199.188.74.124
addnode=162.194.217.242
server=1
listen=1
EOF

chown -R monstercoin:monstercoin /home/monstercoin/.monstercoin

# build MonsterCoin
make -f makefile.unix

cp ./monstercoind /opt/
chmod 755 /opt/monstercoind

# create a monstercoin service
cat << EOF > /etc/systemd/system/monstercoin.service
[Unit]
Description=monstercoin
After=network.target

[Service]
Type=simple
User=monstercoin
Group=monstercoin
WorkingDirectory=/opt/
ExecStart=/opt/monstercoind -debug -printtoconsole
Restart=on-abort

[Install]
WantedBy=multi-user.target
EOF


# run monstercoin wallet as a service
systemctl daemon-reload
systemctl enable monstercoin
systemctl start monstercoin

#####
# begin building miner
#####
cd ~/
git clone https://github.com/pooler/cpuminer.git
cd cpuminer
apt-get install libcurl3 libjansson-dev autotools-dev automake autoconf m4 libtool pkg-config libcurl-openssl1.0-dev
./autogen.sh
./nomacro.pl
./configure CFLAGS="-O3"
make

# install cpu miner
cp ./minerd /opt/
chmod 755 /opt/minerd

# Create miner service
cat << EOF > /etc/systemd/system/monstercoin-miner.service
[Unit]
Description=monstercoin-miner
After=network.target

[Service]
Type=simple
User=monstercoin
Group=monstercoin
WorkingDirectory=/opt/
ExecStart=/opt/minerd -o http://localhost:9332 -Omonstercoinrpc:C2xr7dcABXAmhvidci7Q24LNR1VzRVyRpWWLtEqk9k8P -t 4
Restart=on-abort

[Install]
WantedBy=multi-user.target
EOF

# start the miner
systemctl daemon-reload
systemctl enable monstercoin-miner
systemctl start monstercoin-miner
```


License
-------

monsterCoin is released under the terms of the MIT license. See `COPYING` for more
information or see http://opensource.org/licenses/MIT.

