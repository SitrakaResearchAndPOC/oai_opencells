# oai_opencells
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
```
echo -n | openssl s_client -showcerts -connect gitlab.eurecom.fr:443 2>/dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' | sudo tee -a /etc/ssl/certs/ca-certificates.crt
```
```
sudo apt-get install libboost-all-dev libusb-1.0-0-dev python-mako doxygen python-docutils python-requests python3-pip cmake build-essential
```
```
pip3 install mako numpy
```
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
Patch for opencells
```
cd ~
```
```
wget https://open-cells.com/opencells-mods-20190923.tgz
```
```
tar xf opencells-mods-20190923.tgz
```
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
Reload privilege tables now: yes </br>




# Link
* [open_cells](https://open-cells.com/index.php/2019/09/22/all-in-one-openairinterface/)
