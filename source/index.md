## GPG Cheatsheet



GPG stands for GNU Privacy Guard and it is vital tool for everyone who wants to protect the email and files. Like it's inspiration PGP, GPG uses public-key cryptography where each user owns two keys: for decryption (public key) and for encryption (private key).

This cheatsheet is for quick reference and gives a brief description of that could be done with this security tool.

<hr>

#### Keys management

GPG has several switches that allows to generate a public-private keypar, revoke and delete keys. The viewing public and private keys are also supported.

<hr>

#### Generate key pair

To generate a public-private key pair in the GPG keyring use the –gen-key switch. The below command will prompt for some details such the key-type, key-size, user ID to identify the key and the time period over which the generated key will be valid:

##### gpg --gen-key

<hr>

#### Revoke keys

In case if the secret key has been stolen, an existing key could be revoked with –gen-revoke switch. To be able to do this, is is needed for a secret key:

##### gpg --gen-revoke

<hr>

#### Delete keys

The below commands are used to delete public and private keys from GPG keyring:

##### gpg --delete-key KeyID
##### gpg --delete-secret-key KeyID
##### gpg --delete-secret-and-public-key KeyID

<hr>

#### View keys

To view all types of keys, issue the command:

##### gpg --list-key
##### gpg --list-public-keys
##### gpg --list-secret-keys

<hr>

#### Keys manipulations

GPG allows several key manipulation functions such importing and exporting the keys. Sending and receiving keys from server are also supported.

<hr>

#### Export a private key to a file

##### gpg --export-secret-key -a > file.out

<hr>

#### Export a public key in a file

##### gpg --export --armor KeyID > file.out

<hr>

#### Import a private key from a file

##### gpg --import --allow-secret-key-import file.in

<hr>

#### Import a public key from a file

##### gpg --import file.in

<hr>

#### Send a public key to a server

##### gpg --keyserver dbma.keyserver.ca --send-key KeyID

<hr>
#### Get a Key from a server

##### gpg --keyserver dbma.keyserver.ca --recv-keys KeyID

<hr>

#### Cryptographic options

GPG allows file encryption and decryption. Another option is support for file signing that makes this program an alternative to PGP.

<hr>

#### Encrypt, decrypt and sign

##### gpg -r KeyID -e -a -o file.out file.in

<hr>

#### Decrypt file

##### gpg -r KeyID -d -o file.out file.in

<hr>

#### Create signature of a file

##### gpg -b -a KeyID -o file.out file.in

<hr>

##### Also, you might check the GUI frontends for GnuPG for easy GPG management
