`SFTP <https://en.wikipedia.org/wiki/SSH_File_Transfer_Protocol>`_, which stands for SSH File Transfer Protocol or Secure File Transfer Protocol. 
This is a secure version of `FTP <https://en.wikipedia.org/wiki/File_Transfer_Protocol>`_ that uses 
`SSH <https://en.wikipedia.org/wiki/SSH_(Secure_Shell)`_ to transfer files remotely. 
The IT department prefers, for the sake of security, to use the SFTP protocol instead, due to security vulnerabilities. 

In order to use SSH, the users need to have a have generated an SSH key. 
The public key of the client needs to be on the list of known hosts for the server. 
If not, the client will not be able to authenticate their SSH connection with the server. 
In order to design for reuse, it is wise to create a service account with its own SSH keys, 
to perform the automated data ingestion, rather than exposing a the public key of an individual user.

The `OpenSSH <https://www.openssh.com/>`_ software package for Linux systems, 
allows us to perform all the necessary steps required to establish an SSH connection. 
`Here <https://www.ssh.com/ssh/copy-id>`_ is the tutorial for that process.

Generate SSH Keys
~~~~~~~~~~~~~~~~~

First we must generate an SSH key. This is stored in the hidden folder `.ssh` in the home directory `home/<username>` of your user. 

.. code-block:: bash 

    ssh-keygen

Then we must copy our public key to the server. The copying may ask for a password. 

.. code-block:: text 
    
    $ ssh-copy-id -i ~/.ssh/mykey user@host

Now we can test our key has been created by trying to SSH to the server. This will automatically look in our `.ssh` folder if it exists, for our public key.

.. code-block:: text 
    
    $ ssh user@host

We only ever send our public key to the server, we keep our private key a secret, and do not share it with anyone. Treat this like a password.

SFTP Linux Command
~~~~~~~~~~~~~~~~~~

`sftp` is a command that can be used directly in the Linux terminal. [
`Tutorial <https://linuxize.com/post/how-to-use-linux-sftp-command-to-transfer-files/>`_

To connect to a SFTP server simply execute this:

.. code-block:: text 

    $ sftp remote_username@server_ip_or_hostname

If the ssh keys have been generated correctly and copied to the server the user should not be prompted for a password.

Once connected to the server, it functions very similarly to the `ftp` command. 
Typing `help` will produce a list of all the possible commands and their description.

To download a file, say `example.txt` from the server to the client, execute the following command:

.. code-block:: text 
    
    sftp> get example.txt

Then to close the SFTP connection the user must say good bye.

.. code-block:: text 
    
    sftp> bye

WinSCP 
~~~~~~~

TODO:
    * Expand \w screenshots. 
    * Use Paegent for ssh-rsa key. 
    * My SSH key works for NZODN. 
    * Use WinSCP to connect to the server.
    * Visualization of directory structure.

Glossary
~~~~~~~~

FTP
^^^

`FTP <https://en.wikipedia.org/wiki/File_Transfer_Protocol>`_ is the standard network protocol that is used to transfer files from a server to a client. 
It is not considered a secure form of communication, since although it can be configured to use a username and password for each client, 
this is vulnerable to brute force dictionary and social engineering attacks. 

SSH
^^^

`SSH <https://en.wikipedia.org/wiki/SSH_(Secure_Shell)>`_ relies on a two-way handshake between the server and the client. 
The handshake is established using public and private key pairs. The server and the client share their public keys, and keep their private key a secret. 
The private key of the client, and the public key of the server, can be combined to form a send a connection request to the server. 
The server then uses its private key, and the clients public key, to authenticate the request. And so, a secure shell connection can be formed.