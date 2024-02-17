# oai_opencells
## Installing git
```
sudo apt update
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
git clone https://github.com/EttusResearch/uhd.git
```
```
cd uhd
```
```
git checkout v3.15.0.0
```
```
cd ..
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
reboot
```
```
sudo uhd_images_downloader
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
cd ..
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
USE mysql;
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
sudo systemctl restart mysql.service
```
```
sudo mysql_secure_installation
```
The last command will ask a few questions: </br></br>
password: set your password (linux is set in our default config files) </br>
DEFAULT PASSWORD : linux </br>
VALIDATE PASSWORD PLUGIN: no </br>
Remove anonymous users: yes </br>
Disallow root login remotely: yes </br>
Remove test database and access to it: yes </br>
Reload privilege tables now: yes </br> </br>

## Installing MME
```
cd
```
```
cd openair-cn; source oaienv; cd scripts
```
```
gedit ../build/tools/build_helper
```
goto line 330 replace : git://git.osmocom.org/libgtpnl by https://git.osmocom.org/libgtpnl </br></br>
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
 ./build_spgw -i
```
Do you want to install libgtpnl ? <y/N>: y </br>

## Compile the EPC nodes
```
cd
```
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
cd
```
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
source oaienv  
```
```
./cmake_targets/build_oai -I  # install SW packages from internet
```
```
./cmake_targets/build_oai -w USRP --eNB --UE # compile eNB and UE
```
## Our Network setup description
I’ve made a simple configuration for this all-in-one setup. </br></br>
Each node is on a separate IP address, this address is used for all it’s interfaces. In our case of all-in-one, we take addresses on the loopback: this will be fine on all your machines.</br></br>

HSS is on localhost: 127.0.0.1 </br>
eNB is on 127.0.0.10 </br>
MME is on 127.0.0.20 </br>
SPGW is on 127.0.0.30 </br>
The LTE diameter configuration is now isolated from Linux hostname. </br></br>

realm for our EPC: “OpenAir5G.Alliance”, so, full distinguish names (FQDN) are: hss.OpenAir5G.Alliance, mme.OpenAir5G.Alliance </br>

## Install this configuration for eNB

In your eNB configuration file, the network is now fixed, as lo interface always exists and our computer internal addresses also: </br>
```
////////// MME parameters:
 mme_ip_address = ( { ipv4 = "127.0.0.20";
 ipv6 = "192:168:30::17";
 active = "yes";
 preference = "ipv4";
 }
 );

NETWORK_INTERFACES : 
 {
 ENB_INTERFACE_NAME_FOR_S1_MME = "lo";
 ENB_IPV4_ADDRESS_FOR_S1_MME = "127.0.0.10/8";

 ENB_INTERFACE_NAME_FOR_S1U = "lo";
 ENB_IPV4_ADDRESS_FOR_S1U = "127.0.0.10/8";
 ENB_PORT_FOR_S1U = 2152; # Spec 2152
 };
```

In the eNB config file, you need also to set the MCC and MNC as per your SIM card: </br> </br>

tracking_area_code = “1”; </br>
mobile_country_code = “208”; </br>
mobile_network_code = “92”; </br> </br>

And obviously, your radio parameters. </br> </br>

Wwe tested with USRP B210 20MHz band, Huawei E3272 UE, a cavity duplexer a simple antenna, about 1 meter distance UE/eNB antenna with this file: ~/opencells-mods/enb.10MHz.b200 </br></br>

if you use the OpenAir UE, a sim card file that match our hss database example: opencells-mods/sim.conf. We will make another tutorial to use together OpenAir UE and rf board simulation </br>

## Install this configuration for EPC
For the EPC, we install in OAI default directory: /usr/local/etc/oai </br> </br>
```
sudo mkdir -p /usr/local/etc/oai
```
```
sudo cp -rp ~/opencells-mods/config_epc/* /usr/local/etc/oai
```
```
cd 
```
```
cd openair-cn; source oaienv; cd scripts
```
```
./check_hss_s6a_certificate /usr/local/etc/oai/freeDiameter hss.OpenAir5G.Alliance
```
```
./check_mme_s6a_certificate /usr/local/etc/oai/freeDiameter mme.OpenAir5G.Alliance
```

Only the SGi output to internet need to be configured. </br>
In /usr/local/etc/oai/spgw.conf, </br>
your should set the Ethernet interface that is connected to Internet, and, </br>
to tell to the PGW to implement NAPT for the UE traffic </br> </br>
```
 PGW_INTERFACE_NAME_FOR_SGI = "enp3s0"; 
 PGW_MASQUERADE_SGI = "yes";
```

For the SIM card, you’ll have more to do: </br></br>

SIM MCC/MNC should be duplicated in a couple of files </br>
eNB: See above in eNB configuration chapter </br>
MME file: /usr/local/etc/oai/mme.conf to update </br>
```
GUMMEI_LIST = ( MCC="208" ; MNC="92"; MME_GID="4" ; MME_CODE="1"; } );
TAI_LIST = ({MCC="208" ; MNC="92"; TAC = "1"; } );
```
* HSS
Configure the password for MySQL </br>
in /usr/local/etc/oai/hss.conf, set password as the password you created during MySQL installation </br>
* A HSS database in text is in: ~/opencells-mods/opencells_db.sql
We don’t use phpmyadmin: we load the database from a ascii file </br>
It is pre-configured with the mme id </br>
10 users is network 208/92 (a French test network) are also created (don’t use 3GPP test network: 001/01: the mme fails when MCC starts by “0”) </br>
* Each time you import this db, it erases the entire database
(example: you set mysql password to “linux”) </br>

```
~/opencells-mods/hss_import 127.0.0.1 root linux oai_db ~/opencells-mods/opencells_db.sql
```
We use to modify the db by updating this file with regular text editor, </br>
then we re-load the entire database, </br>
but, if you prefer, usage of http://localhost/phpmyadmin is fine. </br>
if you modified the hss db directly, we offer a export script: </br>
```
~/opencells-mods/hss_export
```
The important values to set are: </br>
table pdn: </br>
all IMSI are listed, with the APN: these values are in UE/USIM </br>
table users: </br>
all IMSI, key (Ki) and OPc must be the same in USIM card </br>
Sqn increments automatically  when the UE authenticate in both USIM and HSS DB: it should be set as per USIM internal incrementation </br>
SIM card update </br>
Open cells UICC and card reader will  be supported </br>

## Final test and verification

open 4 terminal windows </br>
in each window
* Terminal 1
```
cd
```
```
cd openair-cn; source oaienv; cd scripts; ./run_hss
```
* Terminal 2
```
cd
```
```
cd openair-cn; source oaienv; cd scripts; ./run_mme
```
* Terminal 3
```
cd
```
```
cd openair-cn; source oaienv; cd scripts; sudo -E ./run_spgw
```
* Terminal 4
```
cd
```
```
sudo bash
```
```
cd ~/openairinterface5g; source oaienv
```
```
cd cmake_targets/lte_build_oai/build
```
```
./lte-softmodem -O ~/opencells-mods/enb.10MHz.b200
```

Connect the UE, it should attach to network and be able to reach internet through OAI network. </br></br>

If the UE attaches, but you don’t have internet access, verify phone configuration: enable data in config->sim and verify the APN value </br> </br>

Issues related to CPU power </br>
If you reach performance issues: USRP/UHD prints “LLLLL” or the process exits “problem with samples”, OVERFLOW, … </br> </br>

The first case is to verify the USRP dialogs over USB3 (not USB2): the process must report: </br>
```
Found USRP B200
-- Detected Device: B200
-- Operating over USB 3.
```
For OAI source code, we wrote improvements and some hints for UE performance last year. The Linux/Ubuntu advises can be applied to the eNB: </br> </br>

https://gitlab.eurecom.fr/oai/openairinterface5g/wikis/setup-for-real-time-performance

We may make later a post for eNB (OAI/eNB can reach much better performance than today develop branch, but it require to enhance parts of the source code). </br> </br>

We re-built this procedure from scratch and tested i5-6600K. </br> </br>

On the i5-6600K, we obtained stable performance at maximum traffic over 20MHz, transmission mode 1 (SISO), duplexer, antenna, E3372 UE (one single UE) over-the-air 10 cm distance : </br>

# Link
* [open_cells](https://open-cells.com/index.php/2019/09/22/all-in-one-openairinterface/)
