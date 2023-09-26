Ref: https://www.youtube.com/watch?v=rt6Z_ph8ZAg
Ref:https://www.youtube.com/watch?v=106uH7USwZ8

_Note: Using Ubuntu_
### Install JDK 11
Run this command from your home directory:
`sudo apt-get install -y openjdk-11-jdk`

### Install Ghidra
Get the link of the zip under:
https://github.com/NationalSecurityAgency/ghidra/releases
1. `wget [link to zip file]`

2. `unzip [name of the file]`

3. _Optional:_ `mv [name of the unzipped file] [ghidra]`

4. `cd ghidra`

### Run ghidra
Note: There is a way to create a desktop icon, which was skipped in this tutorial.
To run ghidra, using the ghidraRun file, input the following:
`./ghidraRun`


## Troubleshooting
### Ghidra won't find Java Path automatically
Follow these steps if ghidra won't find the jvm automatically:
1. check if java is installed: `java -version`
2. check for the location of the jvm: `whereis jvm`
3. copy the location: (in my case) `/usr/lib/jvm/java-11-openjdk-amd64`
4. open with the editor of your choice: `nano ~/.bashrc`
4.1. add the following lines:
```
#JAVA_HOME
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64

export JAVA_HOME

export PATH=$PATH:$JAVA_HOME/bin
```
5.  Close the terminal or use a new terminal window and input :`echo $JAVA_HOME`
6.  If it worked, you should get:
```
fabio@fabio-VirtualBox:~$ echo $JAVA_HOME
/usr/lib/jvm/java-11-openjdk-amd64
```
