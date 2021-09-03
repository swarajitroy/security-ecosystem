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



## The Application 
---


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


