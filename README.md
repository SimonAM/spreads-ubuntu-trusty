#Installing spreads for ubuntu 14.04 from scratch
the exact spreads git revision i am using here in the tutorial is:
[3a2b9e075c3de0fa9dc81921d598ac559fdc87b3](https://github.com/DIYBookScanner/spreads/commit/3a2b9e075c3de0fa9dc81921d598ac559fdc87b3)

this documentation shows howto install [spreads](https://github.com/DIYBookScanner/spreads/) for ubuntu 14.04. It's heavily based on the following resources:

##sources:
[readthedocs](http://spreads.readthedocs.org/en/latest/)

[SIUA installation guide](https://github.com/LabExperimental-SIUA/ilt/wiki/Spreads-Installation---Configuration-Guide-%28Debian-Jessie-Sid%29)

[important diybookscanner forum entry](http://www.diybookscanner.org/forum/viewtopic.php?f=35&t=3033)


starting from a clean 14.04 system

```bash
sudo apt-get update
sudo apt-get upgrade	
```
afterwards I did a restart (I always do so when kernel is upgraded)

now install all dependcies

```bash
sudo apt-get install python2.7 python2.7-dev python-virtualenv \
python-tk idle python-pmw python-imaging \
python-pip libpython2.7-dev libusb-dev \
libjpeg-dev libtiff5-dev libqt4-core ruby ruby-dev ruby-rmagick \
libmagickwand-dev \
 ruby-hpricot scantailor lua5.2 git 
```

next install pdfbeads

```bash
sudo gem install pdfbeads
```

next install latest djvubind (for me was 1.2.1)

```bash
wget http://djvubind.googlecode.com/files/djvubind_1.2.1.deb
sudo dpkg -i djvubind_1.2.1.deb
sudo apt-get -f install
sudo dpkg -i djvubind_1.2.1.deb
```

Next install latest chdkptp. Go to website, for me it was version v650 (chdkptp-r650-Linux-x86_64.zip), from [www.assembla](https://www.assembla.com/spaces/chdkptp/documents)
unzip to /usr/local/lib/chdkptp

```bash
sudo unzip chdkptp-r650-Linux-x86_64.zip -d /usr/local/lib/chdkptp
```

install latest canvas draw, mine was 5.8
```bash
wget http://downloads.sourceforge.net/project/canvasdraw/5.8/Linux%20Libraries/cd-5.8_Linux35_64_lib.tar.gz
sudo tar zxf cd-5.8_Linux35_64_lib.tar.gz -C /usr/local/lib/chdkptp
```

next install IUP - portable user interface in the latest version, mine was 3.11
```bash
wget http://downloads.sourceforge.net/project/iup/3.11/Linux%20Libraries/iup-3.11_Linux35_64_lib.tar.gz
sudo tar zxf iup-3.11_Linux35_64_lib.tar.gz -C /usr/local/lib/chdkptp
```
add the /usr/local/lib/chdkptp path to the systems dynamic library search path

create and open a new file
```bash
sudo nano /etc/ld.so.conf.d/spreads.conf
```
add the line (And then save with ctrl+o, exit with ctrl+x)
```bash
/usr/local/lib/chdkptp/
```
reload the system-wide libraries paths
```bash
sudo ldconfig
```
now install libyaml

```bash
wget http://pyyaml.org/download/libyaml/yaml-0.1.5.tar.gz
tar xvf yaml-0.1.5.tar.gz
cd yaml-0.1.5
./configure
make 
sudo make install
```

finally install spreads in an virtualenv, create a new one

```bash
virtualenv ~/.spreads
source ~/.spreads/bin/activate
```

install further dependancies for spreads (ignore warnings)
```bash
sudo apt-get install python-psutil libffi-dev  python-usb libturbojpeg
```

fix problems with the libturbojpeg dyn lib

```bash
sudo ln -s /usr/lib/x86_64-linux-gnu/libturbojpeg.so.0.0.0 /usr/lib/x86_64-linux-gnu/libturbojpeg.so

```
install further python modules for spread
```bash
pip install pycparser 
pip install cffi 
pip install jpegtran-cffi

```
update for python usb
```bash
pip install --upgrade --pre pyusb
```

enable spreads GUI packages by installing PySide and fixing symbolic link problem

```bash
sudo apt-get install python-pyside
sudo ln -s /usr/lib/python2.7/dist-packages/PySide ~/.spreads/lib/python2.7/site-packages/PySide
```

add current user to staff group  (the word ´username´ must be replaced by the current username)

```bash
sudo adduser username staff
```

now add the lua env variable to the global path in order that the chdkptp command will work
open the file (or press ctr+h, in correct folder to find the file. And enter the lines in the correct place, then save)
```bash
vi ~/.bashrc 
```
add
```bash
export CHDKPTP_DIR=/usr/local/lib/chdkptp
export LUA_PATH="$CHDKPTP_DIR/lua/?.lua"
source ~/.spreads/bin/activate
```

open a new shell or type 
```bash
source ~/.bashrc
```
Install som further dependencies.
```bash
sudo apt-get install liblua5.2-dev
pip install --install-option='--no-luajit' lupa
pip install chdkptp.py
```
we need some more python modules for the spread web plugin
```bash
pip install Flask
pip install tornado
pip install requests
pip install waitress
pip install zipstream
pip install Wand
pip install Flask-Compress
```

Now install spreads by checking out the github repo. First compile the web client, then ´pip install´ the rest.
We have to do this because of a bug in the current web client (as of revision [3a2b9e075c3de0fa9dc81921d598ac559fdc87b3](https://github.com/DIYBookScanner/spreads/commit/3a2b9e075c3de0fa9dc81921d598ac559fdc87b3), see issue [126](https://github.com/DIYBookScanner/spreads/issues/126))
First we need to install the node.js program, then we change the makefile for the client because the tests do not work at the moment and will therefore not allow to complete compilation.
We then compile the client using the modified makefile.
Then we install spreads with the compiled client,

install node.js
```bash
curl -sL https://deb.nodesource.com/setup | sudo bash -
sudo apt-get install nodejs
```

now change the makefile, compile the client and install spreads with the manual-compiled client 
```bash
cd /tmp
git clone https://github.com/DIYBookScanner/spreads.git
sed -Ei 's/(all: )test (development production)/\1\2/g' spreads/spreadsplug/web/client/Makefile
cd  spreads/spreadsplug/web/client
make clean
make all
cd /tmp/spreads
pip install .
```
If you are scanning books in non-english languages. Find the corresponding language file for text recognition. 
The Swedish tesseract file is called tesseract-ocr-swe. 
So swedes would use...
```
sudo apt-get install tesseract-ocr-swe
```

now run the spreads configuration program

```sh
spread configure
```
in the program set

* step 1 select : chdkcamera
* step 2 select : scantailor, tesseract, gui, autorotate, web
* step 3 select : select order - autorotate,scantailor,tesseract 
* step 4 and 5 select : no for both camera questions (target_page and focus)

this will save the config as config.yaml.

now its time to configure the cameras

Before rerunning the spreads config program again first make sure if you can use the 
chdkptp program.

First before using chdkptp in Ubuntu 14.04 you have to kill gphoto in order to gain access to the chdk-enabled cameras otherwise you will not be able to use them (gphoto2 will block access to the cameras for all other processes). The first number to the left is the PID, this has to be inserted into the third line in the following code. 
```bash
ps aux | grep gphoto
kill -9 PID_of_gphoto2_process
```

I use the Canon A2200, enable CHDK in your camera 
(click [here](http://chdk.wikia.com/wiki/CHDK_1.2.0_User_Manual) - section Using CHKD), plug in the micro USB after 
enabling (I use firmware update method) then plug in the other USB end into your computer, then run the
chdkptp program to test if it is installed correctly.

```bash
/usr/local/lib/chdkptp/chdkptp
```
now click on Connect button.

If it is running without problems end the program.

Now restart the spread configure program again, enable CHDK in your camera, then connect to the computer
 (if not done already) and configure target_page and focus. Should give no errors if camera and CHDK has been 
setup correctly.
