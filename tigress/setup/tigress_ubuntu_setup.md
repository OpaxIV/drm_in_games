## Pre-Installation
*Ubuntu was used for this installation*
1. Input command `sudo apt-get update && upgrade`
2. On Linux, we provide a .deb package. Download, and, in a Terminal, type `sudo dpkg --force-architecture -i  tigress_3.3.3-1_any.deb`. Tigress will be installed in /usr/local/bin/tigresspkg and links to the tigress binaries will be added to /usr/local/bin.
3. Install `sudo apt-get install jq` 
4. Run the installation check: `/usr/local/bin/tigresspkg/3.3.3/check.sh`.

## Installation from the Terminal
1. Input the following commands into the shell:
```
export TIGRESS_HOME=/PATH_TO/tigress/VERSION
export PATH=$PATH:/PATH_TO/tigress/VERSION
```
In my case:
```
export TIGRESS_HOME=/usr/local/bin/tigress/3.3.3
export PATH=$PATH:/usr/local/bin/tigress/3.3.3
```
2. 

## Usage
*In your file, which you want to obfuscate, add the following header line: `#include "PATH-TO-TIGRESS-INSTALLATION/tigress.h"`, in my case `#include "/usr/local/bin/tigresspkg/3.3.3/tigress.h"`*

## Troubleshooting
- If in any case `Cannot start GNUCC at /usr/local/bin/tigresspkg/3.3.3/Linux-x86_64/App/Cilly.pm line 2212.` appears as an error, make sure to update and upgrade your system and to have gcc installed.

