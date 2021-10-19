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
