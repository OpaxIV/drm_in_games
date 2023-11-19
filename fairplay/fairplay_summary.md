# Summary of the FairPlay Article
_URL: https://nicolo.dev/en/blog/fairplay-apple-obfuscation/_
<br>

## Table of Contents
1. [Foreword](#foreword)
2. [DRM systems and the protection of Apple applications](#drmsystems)
3. [Static analysis and obfuscation techniques](#staticanalysis)
4. [The Fairplayd Daemon](#fairplaydeamon)
<br>

## Foreword <a name="foreword"></a>
- FairPlay is used by Apple as a tool in the digital rights management.
- Handles the decryption of iOS applications during their installation on Apple devices.
- Apple distributes all applications in the Apple Store through the IPA file format.
- The IPA file format contains encrypted information that will then be used by the operating system to install an application.
- All of the encrypted information is handled through FairPlay, which takes care of keeping the decryption key and the whole process secure to avoid the possibility of decrypting the contents of.ipa files to share the contents of an app (perhaps paid for) in the wrong hands.
<br>

## DRM systems and the protection of Apple applications <a name="drmsystems"></a>
### DRM Systems in General
- Receive as input a secret (e.g. film, image, or algorithm) that is not readable.
- For example raw files then process the information and transmit as output the original content.
- All this is done while trying to keep the process by which the original information was obtained as hidden as possible.
- DRM systems are most widely used for viewing copyrighted content: user signs a contract with a service provider that promises to send the requested information (e.g. movies, TV series, books).
- To prevent copying of the content (the contract provides for only one user authorized to view it with that contract), the information must be accessible so that no copying or exporting of the content is expected.
### Apples DRM System
- Uses dynamic keys for
- Generation of “dynamic” keys, i.e., depending solely on the installation device, the account of the user who paid for the applications, and a set of metadata exchanged during the transaction (to avoid spoofing).
- Apple encrypts the contents of the IPA file the moment it receives a new request from the Apple Store: encryption is done with a public key associated with the Apple account. Once received, the archive is decompressed and the individual binary is decrypted using the private key within the device.
- The application is thus installed in plaintext and remains in plaintext within the Apple device.
- Apple uses a wide array of authentification parameters like the Apple Store, iCloud and Apple Signing.
- FairPlays usage is not limited to iOS (e.g. MacOS)
<br>

## Static analysis and obfuscation techniques <a name="staticanalysis"></a>
- Protecting how the decryption process works is the main goal of another set of “anti-reverse engineering” technologies named software obfuscation.
- Procedure called static analysis that allows for more specific investigation of certain parts of a piece of software without sending it into execution.The main technique of static analysis is reverse engineering, which is the reconstruction of the original code by inferring information from raw binary code.
- Raw binaries in fact consist of two main pieces of information to function: data and instructions. Through a multitude of steps, programs such as Ghidra, IDA or Binary Ninja can reconstruct much of the original source code. Although not perfectly, the software analyst is able to infer much of the semantics of the software: how it works, what methods it calls, and what information it uses from the operating system are some of the examples of questions we can answer through reverse engineering.
- Reverse engineering allows one to derive to a good degree of approximation the algorithms that FairPlay uses in order to decrypt the content.
- The solution to protect the instructions and data is to apply some forms of obfuscation that make the process of reverse engineering analysis more difficult (hence the use of FairPlay)
<br>

## The Fairplayd Daemon <a name="fairplaydeamon"></a>
### CoreFP.Framework
- MacOS: Much of the management of FairPlay is assigned to a framework called CoreFP.Framework (Core Fair Play) found within the /System/Library/PrivateFrameworks folder.
- Private frameworks are a set of libraries dedicated to certain specific macOS features that are considered private (not been released for public use) and all methods contained within are to be considered valid only for Apple applications.
- Used by some applications and daemons such as Safari and AMPLibraryAgent.
- Command to check where a private framework is currently being used: `lsof | grep -i [name_framework]` where name_framework is the name of the framework library.
- As an example the private framework AMPLibraryAgent:
  - User-space daemon used to manage the user’s media (TV.app and Music.app).
  - A kind of intermediate process between the encrypted content coming from Apple’s servers and the end user interacting with the decrypted content through the TV.app and Music.app clients.
- FairPlayd is the daemon that is invoked by AMPLibraryAgent and is used in practice to decrypt the contents.
- Although the private frameworks are contained within a dynamic cache called dyld, the binaries of CoreFP.Framework are available without any special arrangements that the user has to make on the dynamic cache.
- Contained in the CoreFP.framework are `CoreFP` and `fairplayd`:
  - `CoreFP` is the binary that is used by system processes and constitutes the library.
  - `Fairplayd` is the user space daemon. The user space daemon uses the kernel component called `FairPlayIOKit`.
