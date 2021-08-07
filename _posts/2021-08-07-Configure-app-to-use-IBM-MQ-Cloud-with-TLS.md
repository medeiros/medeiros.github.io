---
layout: post
title: "Configure Java app to use IBM MQ Cloud with TLS"
description: >
  How to configure Java env to use IBM MQ Cloud and TLS.
categories: distributedarchitecture
tags: [java, mq]
comments: true
image: /assets/img/blog/mq/ibm-mq.png
---
> IBM MQ Cloud is the IBM implementation of messaging queue mechanism,
available as a service in the cloud.
In this page, the configuration for its usage and integration with TLS, in the
client perspective, will be covered.
{:.lead}

- Table of Contents
{:toc}

The main goal here is to cover the configuration of IBM Cloud MQ and the
certification burden, so that Java clients can connect and use for dev purposes.
This article do not cover Java client's implementation.
{:.note}

## IBM MQ and Java Developer Perspective

IBM MQ is the IBM implementation of messaging queue mechanism. It is widely
adopted in brazilian companies, and very often it is required to use it in
order to transfer messages between different types of applications.

For the Java developer point of view, a IBM MQ environment is required to
perform tests before go to production. However, depending on the company that
you are, perhaps you may have to create this test environment by yourself.

One alternative is to download and install IBM MQ locally. But, in this case,
one must have admin privileges (there is no such thing as 'portable MQ').

Other alternative is to use IBM MQ as a service, in the cloud. This is a more
simple approach in terms of pre-requirements - and that is the one we'll cover
in this page.

IBM offers 10.000 msg/mo free of charge - ideal for development purposes.
{:.note}

## Create an IBM Cloud account

Go to `http://cloud.ibm.com/` and proceed with the account creation.
You'll get an IBMid, required for authentication.

## Creating a MQ Service

- In the hamburguer menu, select 'Dashboard' and then, click on 'Create Resource'
- Search for 'MQ' in the search bar. Click on the 'MQ' returned option (not
  RabbitMQ or ActiveMQ - just "MQ")
- In the service configuration screen
  - Keep the plan as 'Lite' (to not be charged and for dev purposes)
  - Change the service name to a more descriptive one (like 'MQ-TEST')
  - Click 'Create' in order to create your MQ service

## Create Admin user for MQ Service

- In the Hamburguer menu, click on 'Resource List'
- In 'Services and Software' section, click on MQ-TEST service, previously
created. You are now in the MQ Service screen
- Select the tab 'User Credentials'
- Click 'Add' and create an admin user

## Create a Queue Manager in the MQ Service

- In the MQ Service screen, Click 'Create' button
- Inform settings as required and click 'Create' again to create your Queue
Manager

## Get Queue Manager details

In the Queue Service screen, click on the queue manager name in the datagrid,
and then clck on the 'Administration' tab. Now you can see Queue Manager
required data for external connection, for instance:

```
Hostname - qmanager-123e.qm.us-south.mq.appdomain.cloud
Port - 32354
Queue Channel - CLOUD.ADMIN.SVRCONN
Activate User Identification - Yes
Compatibility Mode for user identification -No (important)
```

The next step is to get an API Key.

## Get Queue Manager API Key

- In the queue manager screen, go to the 'Administration' tab
- Select 'MQ Explorer'
- Click on the 'Rconfigure IBM Cloud API Key'.
- Copy and save this key for later. It will be used, along with admin username,
to perform authentication

And that's it. The next step is to create a TLS certificate.

## Create a Certificate for secure auth

IBM Cloud requires certificate to accept requests.

The idea is to create an local certificate and upload it into IBM Cloud
environment. This certificate will be later used by Java client app to perform
authentication.

### Create local keystore file and convert it into pem file

```bash
keytool -genkey -keyalg RSA -v -keystore mykeystore.jks -alias mykeystore

keytool -importkeystore -srckeystore mykeystore.jks \
   -destkeystore mykeystore.p12 \
   -srcstoretype jks \
   -deststoretype pkcs12

openssl pkcs12 -nodes -in mykeystore.p12 -out mykeystore.pem
```
(based on: https://www.baeldung.com/java-keystore-convert-to-pem-format)

### Create truststore file (based on the keystore)

```bash
keytool -export -alias mykeystore -keystore mykeystore.jks -rfc -file myTrustStore.cert

keytool -import -file myTrustStore.cert -alias myTrustStore -keystore myTrustStore.jks

keytool -importkeystore -srckeystore myTrustStore.jks \
   -destkeystore myTrustStore.p12 \
   -srcstoretype jks \
   -deststoretype pkcs12

openssl pkcs12 -nodes -in myTrustStore.p12 -out myTrustStore.pem
```

### Upload the keystore file into IBM Cloud Queue Manager

- Go to the Queme Manager screen
- Go to the 'Keystore' tab
- Click 'Import Certificate'
- Select your previously created PEM keystore file

## Configure your app to connect to IBM MQ

- Use MQ libraries to create your client. Important informations to collect:
  - MQ: [inform]
  - queue manager: [inform]
  - host: [inform]
  - port: [inform]
  - channel name: CLOUD.ADMIN.SVRCONN
  - queue: [inform]
  - admin username/API key: [inform]
- Add the following JVM parameters to use the created certificates:

``` bash
-Djavax.net.ssl.keyStore=/path/to/mykeystore.jks
-Djavax.net.ssl.keyStorePassword=mykeystore
-Djavax.net.ssl.trustStore=/path/to/myTrustStore.jks
-Djavax.net.ssl.trustStorePassword=mykeystore
```

## References

- https://www.baeldung.com/java-keystore-convert-to-pem-format
- https://www.ibm.com/docs/en/ibm-mq/8.0?topic=support-tlsssl-troubleshooting-information
- https://www.ibm.com/docs/en/ibm-mq/9.2?topic=java-tls-cipherspecs-ciphersuites-in-mq-classes
- https://www.ibm.com/docs/en/ibm-mq/7.5?topic=problems-tlsssl-troubleshooting-information
- https://www.ibm.com/docs/en/ibm-mq/9.0?topic=java-enabling-tls-in-mq-classes
- https://www.ibm.com/docs/en/ibm-mq/9.2?topic=java-tls-cipherspecs-ciphersuites-in-mq-classes
- https://www.ibm.com/support/pages/why-tls-connection-mq-failing-compcode-2-mqccfailed-reason-2400-mqrcunsupportedciphersuite-exception
- https://www.ibm.com/docs/en/ibm-mq/9.1?topic=java-tls-cipherspecs-ciphersuites-in-mq-classes
- https://www.ibm.com/mysupport/s/question/0D50z000062kvHW/what-tls-cipherspecsciphersuites-are-supported-when-connecting-from-oracle-java-nonibm-jre-to-mq-queue-manager?language=en_US
- https://stackoverflow.com/questions/43736957/jmscmq0001-ibm-mq-call-failed-with-compcode-2-mqcc-failed-reason-2400
- https://www.ibm.com/docs/en/ibm-mq/9.1?topic=jms-tls-cipherspecs-ciphersuites-in-mq-classes
- https://stackoverflow.com/questions/55737975/ibm-mq-call-failed-with-compcode-2-mqcc-failed-reason-2035-mqrc-not-au
- https://myshittycode.com/2019/04/23/spring-boot-connecting-to-ibm-mq-over-jms-using-non-ibm-jre/
- https://www.ibm.com/docs/en/ibm-mq/9.1?topic=properties-mq-queue
- https://stackoverflow.com/questions/59773898/ibm-mq-get-message-with-french-symbols
- https://www.ibm.com/docs/en/ibm-mq/7.5?topic=conversion-message-types
- https://www.ibm.com/docs/en/ibm-mq/7.5?topic=messages-mqrfh2-header
- http://www.mqseries.net/phpBB2/viewtopic.php?=&p=171236
- https://www.ibm.com/docs/en/ibm-mq/7.5?topic=applications-creating-destinations#q032240___q032240_4
- https://www.ibm.com/docs/en/rtw/9.2.0?topic=request-adding-message-headers
- https://stackoverflow.com/questions/61340723/why-rfh2-header-is-put-before-the-message-instead-of-in-the-header
- https://stackoverflow.com/questions/51437107/wso2-remove-mqrfh2-header-from-outgoing-ibm-mq-message
- http://www.mqseries.net/phpBB/viewtopic.php?t=38803&sid=1e841dc96cb96b0785397761dc423ef1
- https://www.ibm.com/docs/en/ibm-mq/9.2?topic=applications-creating-destinations-in-jms-application#q032240___q032240_4
