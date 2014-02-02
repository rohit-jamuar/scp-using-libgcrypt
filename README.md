###File encryption/decryption/transmission suite akin to scp (using gcrypt libraries).

####Setup:
1. Primarily, you ought to have the *libgcrypt* library installed on you machine - I used an Ubuntu machine. 
On Ubuntu, open **Terminal** and run
> sudo apt-get install libgcrypt11 libgcrypt11-dev gcc.

2. Once you've installed the library, you can navigate to the folder where you have cloned this repo.
3. Run **make** on **Terminal**

[More on **libgcrypt**](http://www.gnupg.org/documentation/manuals/gcrypt/)

The tool runs in two modes: local (**-l**) and network (**-d**) mode. 

####Working in local mode:
In local mode, you encrypt and store encrypted file(s) locally - in the same path where your source file is present. In local mode, the encrypted file will be stored in the following format : **file_name.gt** (Assuming the source file is named **file_name**.)

In order to encrypt file: 
> ./techrypt *file_name* -l

In order to decrypt file: 
> ./techdec *file_name.gt* -l

####Working in network mode:
In network mode, you must first run the decryption routine - it has to work like a daemon process before it can be in a position to accept request(s) (using **sockets**).

In order to run the decryption daemon:
> ./techdec -d *port_number*

Once the decryption daemon is up, you can transmit the file via:
> ./techrypt *file_name* -d *IP-address*:*port_number*


####Operational details:
1. It is assumed that the decryption routine receives file(s) in the format : *file_name.gt* - they have to end in *.gt*.
2. For decryption routine:

  * In local mode - It is an error if a file, being decrypted, has the same name as the one that would be generated by the decryption routine : for e.g. if *file* exists in the location where the decryption routine is trying to decrypt *file.gt*, the routine will return an exit code **33**.

  * In network mode - It is an error is a file, being received and decrypted, has the same name as a file in the directory from where the decryption routine is running from. This violation will return an exit code of **33**.

2. For the encryption / decryption routine to work properly, you must feed-in the same *passphrase*.
3. A *passphrase* is used to generate a key (using **PBKDF2**) - which is then used for Encryption/Decryption (using **AES128 in CBC mode**) and computing **MAC** (**HMAC** running **SHA512**, internally). Although using the same key in a real setup is a really bad idea, but just for the sake of simplicity, both the encryption / decryption and MAC modules use the same key. Moreover, **IV** values are hard-coded in the program logic - this should be avoided in a real setup as well. Selection of intermediate key(s) and **IV** are usually randomized in real-world implementations.
4. MAC is used for authenticating encrypted file's contents. During encryption phase, **MAC(encrypted content)** is appended to the actual encrypted content. During decryption, MAC value is regenerated and is checked against the stored / transmitted MAC value to ensure that the encrypted content has not altered. If the MACs don't match, the code returns an exit code of **62**.
