# Ubuntu 18.04 downgrade the gcc version to version 5.5

ref:https://gist.github.com/Nannigalaxy/1e473cac77f9f7c8dc4dc3784cc0c2ef#file-downgrade_gcc_version-md

First check your own gcc version, the default is 7.3 on Ubuntu18.04

    $ gcc --version
    gcc (Ubuntu 7.3.0-27ubuntu1~18.04) 7.3.0


1. Download gcc/g++ 5
```
sudo apt-get install -y gcc-5
sudo apt-get install -y g++-5
```
2. Link gcc/g++ to downgrade
```
cd /usr/bin
sudo rm gcc
sudo ln -s gcc-5 gcc
sudo rm g++
sudo ln -s g++-5 g++
```
Check the gcc version again and you can see that it has been downgraded.

    $ gcc --version
    gcc (Ubuntu 5.5.0-12ubuntu1) 5.5.0 20171010

#### To revert this action
```
cd /usr/bin
sudo rm gcc
sudo ln -s gcc-7 gcc
sudo rm g++
sudo ln -s g++-7 g++
```
