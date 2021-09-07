# Create a SAML Application 

## Introduction
---

| Component ID | Component Name | Description|
| -----------  | ----------- |---|
| 01 | SAML IDP (Identity Provider) | Okta|
| 02 | SAML SP (Service Provider) | Spring Security SAML https://spring.io/projects/spring-security-saml|
| 03 | The Application | Web MVC Application using Spring MVC and Thymeleaf |

## Identity Provider (iDP)
---

We will use Okta as our iDP and use Okta developer acccount. We will create a SAML 2.0 application
![Application Type](https://github.com/swarajitroy/security-ecosystem/blob/main/IAM/resources/OktaApplicationType.png)

Following details needs to be provided to create the application - these information comes from Service Provider (SP)

| ID | Details Name | Description|
| -----------  | ----------- |---|
| A | Single Sign On URL or ACS (Assertion Consumer Service) | The application's specific URL that SAML assertions from Okta should be sent to (typically referred to as the ACS). In Okta, this is entered in the application's Single Sign On URL field |
| B | SP Entity ID/Audience Restriction | which dictates the entity or audience the SAML Assertion is intended for. This field is frequently referred to as the "Entity ID" or "Audience URI" by vendors. It can technically be any string of data up to 1024 characters long but is usually in the form of a URL that contains the Service Provider's name within, and is often simply the same URL as the ACS |
| C |  The username format expected by the application | (i.e. email address, first initial last name, etc) |



## Service Provider (SP)
---
Following information is needed from the Identity Provider (iDP) at the Service Provider (SP) side to configure the communication flow.

| ID | Details Name | Description|
| -----------  | ----------- |---|
| A | Identity Provider Single Sign-On URL | The SP may refer to this as the "SSO URL" or "SAML Endpoint." It's the only actual URL Okta provides when configuring a SAML application, so it's safe to say that any field on the Service Provider side that is expecting a URL will need this entered into it |
| B | Identity Provider Issuer | This is often referred to as the Entity ID or simply "Issuer." The assertion will contain this information, and the SP will use it as verification. |
| C |  x.509 Certificate | |

Then we have to include Spring Security SAML https://spring.io/projects/spring-security-saml via Maven dependencies. 

```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.security.extensions</groupId>
    <artifactId>spring-security-saml2-core</artifactId>
    <version>1.0.10.RELEASE</version>
</dependency>

```
Also, make sure to add the Shibboleth repository to download the latest opensaml jar required by the spring-security-saml2-core dependency:

```
<repository>
    <id>Shibboleth</id>
    <name>Shibboleth</name>
    <url>https://build.shibboleth.net/nexus/content/repositories/releases/</url>
</repository>

```



## The Application 
---

The application is a Spring boot MVC application using Thymeleaf. On top of that - the application also bundles the Service Provider logic (Spring Security) into the final JAR file.

```

<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

```

### Service Provider Keystore
---
The service provider has to communicate with Identity Provider in an encrypted channel. 

The Java Keytool is a command line tool which can generate public key / private key pairs and store them in a Java KeyStore. The Keytool executable is distributed with the Java SDK (or JRE), so if you have an SDK installed you will also have the Keytool executable. The Keytool executable is called keytool 


```
C:\pmi>keytool -genkeypair -alias swroyspringsaml -keypass swroysamlokta -keystore saml-keystore.jks
Enter keystore password:
Re-enter new password:
What is your first and last name?
  [Unknown]:  Swarajit Roy
What is the name of your organizational unit?
  [Unknown]:  Security
What is the name of your organization?
  [Unknown]: hello.org
What is the name of your City or Locality?
  [Unknown]:  Kolkata
What is the name of your State or Province?
  [Unknown]:  WB
What is the two-letter country code for this unit?
  [Unknown]:  IN
Is CN=Swarajit Roy, OU=Security, O=ulearnuhelp.org, L=Kolkata, ST=WB, C=IN correct?
  [no]:  yes


Warning:
The JKS keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore saml-keystore.jks -destkeystore saml-keystore.jks -deststoretype pkcs12".
```

## Resource Links
---

- https://www.baeldung.com/spring-security-saml
- https://developer.okta.com/docs/concepts/saml/#authentication
- https://www.samltool.com/online_tools.php
- https://developers.onelogin.com/saml/java
- https://developer.okta.com/blog/2017/03/16/spring-boot-saml
- https://en.wikipedia.org/wiki/SAML_metadata
- https://docs.akana.com/cm/saml/08_glossary.htm
- https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-saml
- https://support.okta.com/help/s/article/Common-SAML-Terms?language=en_US
- https://support.okta.com/help/s/article/Beginner-s-Guide-to-SAML?language=en_US


