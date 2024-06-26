This is Tigress, the C obfuscator, downloaded from tigress.wtf.

This is a fat distribution, with binaries for multiple platforms. Files for each platform go in
   PATH-TO-THIS-DIRECTORY/tigress/3.1/Darwin-x86_64
   PATH-TO-THIS-DIRECTORY/tigress/3.1/Linux-armv7
   PATH-TO-THIS-DIRECTORY/tigress/3.1/Linux-x86_64

## Modifing ~./bashrc file
To install, you need to set the TIGRESS_HOME environment variable:
   export TIGRESS_HOME=PATH-TO-THIS-DIRECTORY/tigress/3.1

You also need to add this directory to your PATH:
   export PATH= ... PATH-TO-THIS-DIRECTORY/tigress/3.1 ...
As such, add the following lines in the `~./bashrc` file:
```
export TIGRESS_HOME=/home/training/Desktop/tigress/3.1
export PATH=$PATH:/home/training/Desktop/tigress/3.1
```

---

## Check Installation
Input the following commands to make sure that the environment variables were added correctly:
```
echo $PATH
echo $TIGRESS_HOME
```
Run the following command to check your install (from your tigress folder):
```
tigress --Environment=x86_64:Linux:Gcc:4.6 --Transform=Virtualize --Functions=main,fib,fac --out=result.c /home/training/Desktop/tigress/3.1/test1.c
```

(Run the following script to check the installation:)
```
PATH TO TIGRESS/TIGRESS VERSION/check.sh
```
---

The tigress script will pick up the right binary to use by trying to figure out which
platform you're on, using "uname -s" and "uname -m". If your uname returns strange
values, this may fail.

To try out Tigress, do
   > tigress --Environment=ENV --Transform=Virtualize --Functions=main,fib,fac --out=result.c PATH-TO-THIS-DIRECTORY/tigress/3.1/test1.c
         where ENV is one of
             x86_64:Linux:Gcc:4.6
             x86_64:Darwin:Clang:5.1
             armv7:Linux:Gcc:4.6 
             armv8:Linux:Gcc:4.6
This should give you an obfuscated program in result.c.

For example, on Darwin you would say:
    > tigress --Environment=x86_64:Darwin:Clang:5.1 --Transform=Virtualize --Functions=main,fib,fac --out=result.c tigress/3.1/test1.c
    > gcc -o result.exe result.c
    > strip result
    > ./result.exe
