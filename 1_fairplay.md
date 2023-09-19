## Résume of the FairPlay Article
#### URL: https://nicolo.dev/en/blog/fairplay-apple-obfuscation/

### Foreword
- FairPlay is used by Apple as a tool in the digital rights management.
- Handles the decryption of iOS applications during their installation on Apple devices.
- Apple distributes all applications in the Apple Store through the IPA file format.
- The IPA file format contains encrypted information that will then be used by the operating system to install an application.
- All of the encrypted information is handled through FairPlay, which takes care of keeping the decryption key and the whole process secure to avoid the possibility of decrypting the contents of.ipa files to share the contents of an app (perhaps paid for) in the wrong hands.


### DRM systems and the protection of Apple applications
#### DRM Systems in General
- Receive as input a secret (e.g. film, image, or algorithm) that is not readable.
- For example raw files then process the information and transmit as output the original content.
- All this is done while trying to keep the process by which the original information was obtained as hidden as possible.
- DRM systems are most widely used for viewing copyrighted content: user signs a contract with a service provider that promises to send the requested information (e.g. movies, TV series, books).
- To prevent copying of the content (the contract provides for only one user authorized to view it with that contract), the information must be accessible so that no copying or exporting of the content is expected.
#### Apples DRM System
- Uses dynamic keys for
- Generation of “dynamic” keys, i.e., depending solely on the installation device, the account of the user who paid for the applications, and a set of metadata exchanged during the transaction (to avoid spoofing).
- Apple encrypts the contents of the IPA file the moment it receives a new request from the Apple Store: encryption is done with a public key associated with the Apple account. Once received, the archive is decompressed and the individual binary is decrypted using the private key within the device.
- The application is thus installed in plaintext and remains in plaintext within the Apple device.
- Apple uses a wide array of authentification parameters like the Apple Store, iCloud and Apple Signing.

- 
