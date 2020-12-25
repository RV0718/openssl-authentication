# openssl-authentication

SSL (Secure Socket Layer) is a protocol used to establish a secure encrypted connection between a server and a client in a network (internet or intranet). Sensitive information can be transmitted securely over an SSL connection. 
A Secure Socket Layer (SSL) certificate is a security protocol which secures data between two computers by using encryption.


## Symmetric Encryption
This is the way to establish a communication between the server and a client using same key i.e. there will be only one key used for encryption and decryption.


![Symmetric Encryption](/Users/RV0718/Study/arch_images/symmetric.jpg "Symmetric Encryption")

![Symmetric Encryption](https://github.com/RV0718/archietecture-images/blob/main/symmetric.jpg "Symmetric Encryption")

Process:   
a. Client will generate the symmetric key using **ssh-keygen** command.   
b. Once done user will now have two files id_rsa and id_rsa.pub.    
c. Now, user will access the secure server from browser.   
d. Browser will render the application page.   
e. Now, user will fill the login page with confidential details like username and password.   
f. Upon click browser/application encrypt that data and send to the server.   
g. In between a hacker sniff the encrypted data, however at this position server and the hacker both can't do anything with this data.   
h. So, user has to share the private generated key with the server, so that server can use the key, decrypt the data and use it.   
i. But, as user sent the key over the internet, hacker can also get the copy of this key and hack your data.   


## Asymmetric Encryption
This is the similar to what we have with symmetric but user will share the symmetric private key with server in a secured way.

Process:
a. Client will generate the symmetric key using **ssh-keygen** command.      
b. Once done user will now have two files id_rsa and id_rsa.pub.   
c. We will generate the server's public, private key and CSR using **openssl** commands explained in below steps.   
d. In this case we will share the server's CSR with authorized CAs to sign it.   
e. CAs used their own different way to sign the certificates.
f. Now, user will access the secure server from browser.   
g. Browser will render the application page.  
h. Server will share the key (CA signed server CSR + public key of the server) with the client.   
i. Now, user will fill the login page with confidential details like username and password.   
j. Upon click, browser/application encrypt that data and send to the server.   
k. Along with this data, client will also send its symmetric private key  by wrapping with server's public key.   
l. Server will use this data and get the client's symmetric key and will use it to decrypt client data.   
m. This way we can setup the secure communication between the server and client.   
n. In this case also hacker can sniff the data and key but can't do anything with that as he does not know how to decrypt the data.   



### 1) Steps to generate the SSL certificates using openssl (a command line tool)

We will be using the openssl way to generate the certificates for both client and server. There are other way also you can use like **EasyRSA** and **CFSSL**.

First we will generate the CA private key and CSR (certificate signing request)  
a. Generate the private key using genrsa command   
  ```sh
  openssl genrsa -out ca.key 4096
  ```
  
b. Now, we will use the generated ca key to create the CSR with all the necessary details   
  ```sh
  openssl req -new ca.key -subj "/CN=localhost" -out ca.csr  
  ```
  
  Here, we have used the **CN as localhost** but you can use the domain name of your application or website. You can also add the system related information like system users, groups etc.
  This command will generate the encoded file containing the public key and information identifying the domain and requestor.
  
c. We will be self signing the CSR (not recommended in production). In production, we won't be doing this way rather we will be sending the CSR to the authorized CA like digicert, symantec. Just for this example, we will be self signing the CSR.   
  ```sh
  openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
  ```
  
d. We will follow the above steps a and b to create the server key and csr by replacing the ca with server.

e. We will sign the server certificate with CA we created, this is what we called as self sign.    
  ```sh
  openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -out server.crt
  ```
  
f. We will convert the certificate into .pem format.
  ```sh
  openssl pkcs8 -topk8 -nocrypt -in server.key -out server.pem
  ```
  

### 2) Verifying using openssl

a. Verify a CSR
  ```sh
  openssl req -text -noout -verify -in server.csr
  ```
  
b. Verify a private key
  ```sh
  openssl rsa -in server.key -check
  ```

c. Verify a certificate
  ```sh
   openssl x509 -in server.crt -text -noout
  ```
  
d. Verify a PKCS#12 file (.pfx or .p12)
  ```sh
   openssl pkcs12 -info -in keyStore.p12
  ```        
  
  
### 3) Debugging using openssl

In case if you're facing any issue while creating the key, or generating the CSR or CRT using private generated key or in signing the CSR, then use the below commands to debug the issues.   
  ```sh
  -  Check an MD5 hash of the public key to ensure that it matches with what is in a CSR or private key
  openssl x509 -noout -modulus -in certificate.crt | openssl md5
  openssl rsa -noout -modulus -in privateKey.key | openssl md5
  openssl req -noout -modulus -in CSR.csr | openssl md5
  -  Check an SSL connection. All the certificates (including Intermediates) should be displayed
  openssl s_client -connect www.paypal.com:443
  ```  
  

##### **Note**: We should never share the private key we generated with anyone.



