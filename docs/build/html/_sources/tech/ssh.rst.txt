`SSH <https://en.wikipedia.org/wiki/SSH_(Secure_Shell)>`_ relies on a two-way handshake between the server and the client. 
The handshake is established using public and private key pairs. The server and the client share their public keys, and keep their private key a secret. 
The private key of the client, and the public key of the server, can be combined to form a send a connection request to the server. 
The server then uses its private key, and the clients public key, to authenticate the request. And so, a secure shell connection can be formed.