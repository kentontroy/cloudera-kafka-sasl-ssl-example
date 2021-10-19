# cloudera-kafka-sasl-ssl-example
Example of how to connect to Kafka hosted by Cloudera CDP using SASL_SSL

```
Nice, healthy log output depicting the successful creation of a Kafka topic with clients authenticating via SASL/GSSAPI (i.e. Kerberos)
and traffic encrypted via TLS 1.2.

21/10/19 20:29:20 INFO utils.Log4jControllerRegistration$: Registered kafka:type=kafka.Log4jController MBean
21/10/19 20:29:21 INFO admin.AdminClientConfig: AdminClientConfig values: 
  ...
  bootstrap.servers = [kafka.cdp.cloudera.com:9093]
  sasl.kerberos.kinit.cmd = /usr/bin/kinit
  sasl.kerberos.service.name = kafka
  sasl.mechanism = GSSAPI
	security.protocol = SASL_SSL
  ssl.enabled.protocols = [TLSv1.2]
	ssl.endpoint.identification.algorithm = https
  ssl.truststore.location = /home/centos/kafka.client.truststore.jks
	ssl.truststore.password = [hidden]
  ...

21/10/19 20:29:21 INFO authenticator.AbstractLogin: Successfully logged in.
21/10/19 20:29:21 INFO kerberos.KerberosLogin: [Principal=kafka/kafka.cdp.cloudera.com@CLOUDERA.COM]: TGT refresh thread started.
21/10/19 20:29:21 INFO kerberos.KerberosLogin: [Principal=kafka/kafka.cdp.cloudera.com@CLOUDERA.COM]: TGT valid starting at: Tue Oct 19 20:29:21 UTC 2021
21/10/19 20:29:21 INFO kerberos.KerberosLogin: [Principal=kafka/kafka.cdp.cloudera.com@CLOUDERA.COM]: TGT expires: Wed Oct 20 20:29:21 UTC 2021

Created topic test.
21/10/19 20:29:22 WARN kerberos.KerberosLogin: [Principal=kafka/kafka.cdp.cloudera.com@CLOUDERA.COM]: TGT renewal thread has been interrupted and will exit.

```
### Step 1 
For each Kafka broker, generate a public/private key pair and certificate persisted in a keystore <br>
The CN should be the FQDN for each broker 
```
keytool -keystore kafka.server.keystore.jks -alias kafka.cdp.cloudera.com -keyalg RSA -genkey

What is your first and last name?
  [Unknown]:  kafka.cdp.cloudera.com
What is the name of your organizational unit?
  [Unknown]:  
What is the name of your organization?
  [Unknown]:  
What is the name of your City or Locality?
  [Unknown]:  
What is the name of your State or Province?
  [Unknown]:  
What is the two-letter country code for this unit?
  [Unknown]:  
Is CN=kafka.cdp.cloudera.com, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown correct?
  [no]:  yes 

Enter key password for <kafka.cdp.cloudera.com>
        (RETURN if same as keystore password):
```
### Step 2
Generate a Certificate Authority (CA) <br>
The CA contains a key pair and a certificate used to sign other certificates
```
openssl req -new -x509 -keyout ca-key -out ca-cert -days 365 

Country Name (2 letter code) [XX]:
State or Province Name (full name) []: 
Locality Name (eg, city) [Default City]:
Organization Name (eg, company) [Default Company Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:kafka.cdp.cloudera.com
Email Address []:
```
### Step 3
Sign the broker's certificate with the CA
```
On each Kafka broker, export the broker's certificate from the keystore: 

keytool -keystore kafka.server.keystore.jks -alias kafka.cdp.cloudera.com -certreq -file cert-file

Sign the certificate with the CA:

openssl x509 -req -CA ca-cert -CAkey ca-key -in cert-file -out cert-signed -days 365 -CAcreateserial

Signature ok
subject=/C=Unknown/ST=Unknown/L=Unknown/O=Unknown/OU=Unknown/CN=kafka.cdp.cloudera.com
Getting CA Private Key
Enter pass phrase for ca-key:

Add the CA's cert to the broker's keystore:

keytool -keystore kafka.server.keystore.jks -alias CARoot -importcert -file ca-cert

Add the signed cert to the broker's keystore:

keytool -keystore kafka.server.keystore.jks -alias kafka.cdp.cloudera.com -importcert -file cert-signed
```
### Step 4
Add the CA's certificate to the truststores on the client (i.e. Kafka Producers/Consumers) and server (i.e. Kafka Brokers)
```
Create a truststore for the clients:

keytool -keystore kafka.client.truststore.jks -alias cdp.cloudera.com -keyalg RSA -genkey

Create a truststore for the the Kafka Brokers:

keytool -keystore kafka.server.truststore.jks -alias kafka.cdp.cloudera.com -keyalg RSA -genkey

Add the CAcert to the truststore:

keytool -keystore kafka.client.truststore.jks -alias CARoot -importcert -file ca-cert

keytool -keystore kafka.server.truststore.jks -alias CARoot -importcert -file ca-cert
```
