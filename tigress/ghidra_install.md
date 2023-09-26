Ref: https://www.youtube.com/watch?v=rt6Z_ph8ZAg

_Note: Using Ubuntu_
### Install JDK 11
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
I tried to work it out but lost my patience.
