# oai_opencells
## Installing git
```
apt update
```
```
sudo apt install git 
```
```
git config --global user.name "Laurent"
```
```
git config --global user.email "laurent.thomas@open-cells.com"
```
## Configuring certificate
```
echo -n | openssl s_client -showcerts -connect gitlab.eurecom.fr:443 2>/dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' | sudo tee -a /etc/ssl/certs/ca-certificates.crt
```
## Installing dependecies for uhd
```
sudo apt-get install libboost-all-dev libusb-1.0-0-dev python-mako doxygen python-docutils python-requests python3-pip cmake build-essential
```
```
pip3 install mako numpy
```
## Installing uhd
```
git clone git://github.com/EttusResearch/uhd.git
```
```
git checkout v3.15.0.0
```
```
cd uhd; mkdir host/build; cd host/build
```
```
cmake -DCMAKE_INSTALL_PREFIX=/usr ..
```
```
make -j4
```
```
sudo make install
```
```
sudo ldconfig
```
```
uhd_images_downloader
```
## Patch for opencells
```
cd ~
```
```
wget https://open-cells.com/opencells-mods-20190923.tgz
```
```
tar xf opencells-mods-20190923.tgz
```
## Installing openair-cn
```
git clone https://gitlab.eurecom.fr/oai/openair-cn.git
```
```
cd openair-cn
```
```
git checkout 724542d0b59797b010af8c5df15af7f669c1e838
```
```
git apply ~/opencells-mods/EPC.patch
```
```
cd openair-cn; source oaienv; cd scripts
```
```
./build_hss -i
```
Answer yes to install: freeDiameter 1.2.0 </br>
phpmyadmin:</br>
We don’t use phpmyadmin later in this procedure to update the MySQL database </br>
We removed the installation of phpmyadmin (of course you can use it if you prefer) </br>
## Configuring database mysql
```
sudo mysql -u root << END
```
```
uninstall plugin validate_password;
```
```
UPDATE user SET plugin='mysql_native_password' WHERE User='root';
```
```
FLUSH PRIVILEGES;
```
```
END
```
```
USE mysql;
```
```
sudo systemctl restart mysql.service
```
```
sudo mysql_secure_installation
```
The last command will ask a few questions: </br></br>
password: set your password (linux is set in our default config files) </br>
VALIDATE PASSWORD PLUGIN: no </br>
Remove anonymous users: yes </br>
Disallow root login remotely: yes </br>
Remove test database and access to it: yes </br>
Reload privilege tables now: yes </br> </br>

## Installing MME
Install 3PP SW for mme and spgw </br>
```
./build_mme -i
```

Do you want to install freeDiameter 1.2.0: no  </br>
Do you want to install asn1c rev 1516 patched? <y/N>: yes </br>
Do you want to install libgtpnl ? <y/N>: yes </br>
wireshark permissions: as you prefer </br>


## Installing SPGW
```
gedit ../build/tools/build_helper
```
goto line 330 replace : git://git.osmocom.org/libgtpnl by https://git.osmocom.org/libgtpnl 
```
 ./build_spgw -i
```
Do you want to install libgtpnl ? <y/N>: y </br>

## Compile the EPC nodes
```
cd openair-cn; source oaienv; cd scripts
```
```
./build_hss
```
```
./build_mme
```
```
./build_spgw
```
If you face compilation issues, the log files are in openair-cn/build/log </br>
In there files, look for “error:” string.</br>

## Download & Compile the eNB on 18.04
```
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
```
```
cd openairinterface5g
```
```
git checkout edb74831dabf79686eb5a92fbf8fc06e6b267d35
```
```


# Link
* [open_cells](https://open-cells.com/index.php/2019/09/22/all-in-one-openairinterface/)
