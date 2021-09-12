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
C:\pmi>keytool -genkeypair -alias swroyspringsaml -keypass **** -keystore saml-keystore.jks -keyalg RSA -keysize 2048 -validity 10000
Enter keystore password:
Re-enter new password:
What is your first and last name?
  [Unknown]:  Swarajit Roy
What is the name of your organizational unit?
  [Unknown]:  security
What is the name of your organization?
  [Unknown]:  abc.org
What is the name of your City or Locality?
  [Unknown]:  Kolkata
What is the name of your State or Province?
  [Unknown]:  WB
What is the two-letter country code for this unit?
  [Unknown]:  IN
Is CN=Swarajit Roy, OU=security, O=abc.org, L=Kolkata, ST=WB, C=IN correct?
  [no]:  yes


Warning:
The JKS keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore saml-keystore.jks -destkeystore saml-keystore.jks -deststoretype pkcs12".

```

### SSO XML
---

The Identity Provider XML is needed. This XML is also known as iDP Metadata and has following 3 critical information for service provider (spring security) to work.

- Identity Provider Entity ID 
- Identity Provider X509 Certificate 
- Identity Provider SSO URL

### Sequence Workthrough
---

| Component ID | Component Name | Description|
| -----------  | ----------- |---|
| 01 | The Spring MVC @Controllers | It uses SecurityContextHolder.getContext().getAuthentication() to ensure pages are protected|
| 02 | The Spring Security SAML Extenstions | Plays the role of a SP Initiated Login/Logout flows |
| 03 | SAML Payload Exchanges with iDP | SAML Request and Response via SAML Tracer Firefox plugin|
| 04 | Additional Assertion to make authorization decisions | |

```
@Controller
public class HomeController {

    @RequestMapping("/")
    public String index() {
        return "index";
    }

    @GetMapping(value = "/auth")
    public String handleSamlAuth() {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        if (auth != null) {
            return "redirect:/home";
        } else {
            return "/";
        }
    }

    @RequestMapping("/home")
    public String home(Model model) {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        model.addAttribute("username", authentication.getPrincipal());
        return "home";
    }

```

```
index.html 

<body>
    <h3><Strong>Welcome to Baeldung Spring Security SAML</strong></h3>
    <a th:href="@{/auth}">Login</a>
</body>
```

```

<saml2p:AuthnRequest xmlns:saml2p="urn:oasis:names:tc:SAML:2.0:protocol"
                     AssertionConsumerServiceURL="http://localhost:8080/saml/SSO"
                     Destination="https://dev-76738796.okta.com/app/dev-76738796_jupiter_1/exk1dw29dzLcYLmvg5d7/sso/saml"
                     ForceAuthn="false"
                     ID="aah6e85079j2cd414h73ej7bih63ih"
                     IsPassive="false"
                     IssueInstant="2021-09-07T15:18:58.266Z"
                     ProtocolBinding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"
                     Version="2.0"
                     >
    <saml2:Issuer xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion">http://localhost:8080/saml/metadata</saml2:Issuer>
    <ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
        <ds:SignedInfo>
            <ds:CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
            <ds:SignatureMethod Algorithm="http://www.w3.org/2000/09/xmldsig#rsa-sha1" />
            <ds:Reference URI="#aah6e85079j2cd414h73ej7bih63ih">
                <ds:Transforms>
                    <ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature" />
                    <ds:Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
                </ds:Transforms>
                <ds:DigestMethod Algorithm="http://www.w3.org/2000/09/xmldsig#sha1" />
                <ds:DigestValue>VL8E38e87I23CkOTLGPUaGFL7P0=</ds:DigestValue>
            </ds:Reference>
        </ds:SignedInfo>
        <ds:SignatureValue>VHbbVA4YcCqMPLKSkHyPCdWibQqsim79Et8AqkpMdGNRLvAjxIEaJWgSTwg/mDiqJEWS3ycXi3MqXSZJ9F0rC6SUnrWHfySajZkYypZ+NurC43qYTSRTNm9NeTTf72xLjSqrk70kzjmiIX8HOViN7at+U/Q/YqKKD2vkG4VuCrQtkRpN0+EMlw8WkFJC47M5LIVePJ2uIDYBESRxYh9iM2TNLLkg57/nnrBepSL7AEXiNv3KFYQd/Dw9Hj9rCjvpz9u/BeR02yBob+jTebQaXA9A6KdEHi6JYqMVlgFGO1UFSBqxDyKbeeR/SMDYyYBQ24a9uygVady9rae1py3KFg==</ds:SignatureValue>
        <ds:KeyInfo>
            <ds:X509Data>
                <ds:X509Certificate>MIIDbzCCAlegAwIBAgIEGiFDNDANBgkqhkiG9w0BAQsFADBoMQswCQYDVQQGEwJJTjELMAkGA1UE
CBMCV0IxEDAOBgNVBAcTB0tvbGthdGExEDAOBgNVBAoTB2FiYy5vcmcxETAPBgNVBAsTCHNlY3Vy
aXR5MRUwEwYDVQQDEwxTd2FyYWppdCBSb3kwHhcNMjEwOTA3MDYxODE2WhcNNDkwMTIzMDYxODE2
WjBoMQswCQYDVQQGEwJJTjELMAkGA1UECBMCV0IxEDAOBgNVBAcTB0tvbGthdGExEDAOBgNVBAoT
B2FiYy5vcmcxETAPBgNVBAsTCHNlY3VyaXR5MRUwEwYDVQQDEwxTd2FyYWppdCBSb3kwggEiMA0G
CSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCH+Tp9WrExuUUrA7nd8RmtsASESMNGtJYtdqZ5mpqH
XepVxJZncQRbGr1wN6bpmiYt/8sTAABFec8dJFLzMGI/NhKHB45/mvGMQUXEq0rRsTpNTN5ij0kn
ib/m4+JssoCbsp9lCgM6i+qoJN+rmVOM/JGfeJ+rzHHQZOnm7PMXaM0oRjAtPlcB1Z6zzPlsXKNh
f6WCRewZifD1PCF8vnxlXPfhNXuWIvFCeJJ6Xcb9PoRecLh+XuAniVfK4i+FqSCZHPWSoorEj5F/
LhsGUaLKbs8WykZive3Dz8JVTKYeY4yChDga5vWelqt1vIDrhhkyLf0LIVBWe3JOOHUUJG0hAgMB
AAGjITAfMB0GA1UdDgQWBBT1PISi5JTdzbch6aif1giBhiiy6DANBgkqhkiG9w0BAQsFAAOCAQEA
TK6J1Fo+F5unS4C13SAr3I5+YUt9XZCmbK9W4x3Vth3CjcBx8pHOFcHwuqsNs6YQiJGO/UJfNAvq
XCxgTEAhIMCVH8vlLigUG6NihocZuay08ZOpAVasl/t3CIrEIGfAucLktTgy7MSi5kRWhXfsK9vC
+wl8zAzjJtdsja3b+tW3NL4oh9FyhNFk363CjKk8Yg/GHMG8QO6gh/fd58W+1+GinMDe2e/kVjwX
aIHbKSCTuaodXR+dDDbdJjrH1UD3GBybyHS99xlT6mv2UKKkAvB0+fqRrMdt8yKMMPVioTvlMaPf
f23tQsAXwYSVf56Amkkhbvm+fqdJpPFh+o0F9w==</ds:X509Certificate>
            </ds:X509Data>
        </ds:KeyInfo>
    </ds:Signature>
</saml2p:AuthnRequest>


```

```
<saml2p:Response Destination="http://localhost:8080/saml/SSO"
                 ID="id16309785466726411066339523"
                 InResponseTo="aah6e85079j2cd414h73ej7bih63ih"
                 IssueInstant="2021-09-07T15:20:10.400Z"
                 Version="2.0"
                 xmlns:saml2p="urn:oasis:names:tc:SAML:2.0:protocol"
                 >
    <saml2:Issuer Format="urn:oasis:names:tc:SAML:2.0:nameid-format:entity"
                  xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion"
                  >http://www.okta.com/exk1dw29dzLcYLmvg5d7</saml2:Issuer>
    <ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
        <ds:SignedInfo>
            <ds:CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
            <ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256" />
            <ds:Reference URI="#id16309785466726411066339523">
                <ds:Transforms>
                    <ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature" />
                    <ds:Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
                </ds:Transforms>
                <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256" />
                <ds:DigestValue>9A/oxcxrOdR+h2NuDkpftfRHYNDO5tr5d/OKcxYKqno=</ds:DigestValue>
            </ds:Reference>
        </ds:SignedInfo>
        <ds:SignatureValue>XTLfdERlP4eCPUlJ8JZq8C7+38v+bl3xI0aCI4pbRaMbEcuxieFkFkVDpKTrMjkg3JVu0iL4z+/Urj6QgwXJ65KTjc642Tf2gd+At6k4fk2XDjx7IQLCiK9+dg9YVMB9YWsfURi9n0b+VVkg0+ttsPluV4t1JuFnVxuN1FFyJ8LmW8DkRlS8Rnr/8hHJ9wzweNQtdBf9P5Tv2BvDIZnnm46hhBMyqzulo0270mXPuRInljkOJ4NViwmFugmAyb72/TDtKoETC8Y43eEM0uir53a4+jhm2ZSfD0NLc3DUtiy94g56B/63P1PPDCx3oVqZFpSl1MYU5lvEwvqzTiMInQ==</ds:SignatureValue>
        <ds:KeyInfo>
            <ds:X509Data>
                <ds:X509Certificate>MIIDqDCCApCgAwIBAgIGAXegb/o5MA0GCSqGSIb3DQEBCwUAMIGUMQswCQYDVQQGEwJVUzETMBEG
A1UECAwKQ2FsaWZvcm5pYTEWMBQGA1UEBwwNU2FuIEZyYW5jaXNjbzENMAsGA1UECgwET2t0YTEU
MBIGA1UECwwLU1NPUHJvdmlkZXIxFTATBgNVBAMMDGRldi03NjczODc5NjEcMBoGCSqGSIb3DQEJ
ARYNaW5mb0Bva3RhLmNvbTAeFw0yMTAyMTQxMjA2MDlaFw0zMTAyMTQxMjA3MDlaMIGUMQswCQYD
VQQGEwJVUzETMBEGA1UECAwKQ2FsaWZvcm5pYTEWMBQGA1UEBwwNU2FuIEZyYW5jaXNjbzENMAsG
A1UECgwET2t0YTEUMBIGA1UECwwLU1NPUHJvdmlkZXIxFTATBgNVBAMMDGRldi03NjczODc5NjEc
MBoGCSqGSIb3DQEJARYNaW5mb0Bva3RhLmNvbTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoC
ggEBAMCpSyeOib6kOTMr3cXwZYVyn39cFohSnF3KaXZrd0MU4mDN5VrDl0oXkvTprPM2H9pL+THa
N3smJUEHhiVd7L6ofePCM1njxfpdagAQbGAJAUUyGIQHRY4qoEBEBA3H9TtSi5HY95TwhwfwC3LW
yhmOKBYJLg5YT2CAziKSHopGSueQiy+mYZSSxM6gWNelZW90Yo4hbptqqB2xhoumQg/jSCubqN49
hQf1pzsFibTOmwG/Jmhm8Jr53Q1A3tAo5vvbST2OPK3IrsQPdDlurMEyCjD2DmTVWFqQwkm8fiN3
cGCvOJSYGaHmDncXCZMpJ4Hc6pbaE9e2Nbw2avvXac8CAwEAATANBgkqhkiG9w0BAQsFAAOCAQEA
nQrbS5ucC9WYX1C4nMcUbaZMkaIEg64OtJbn3SzzfbGABrfKonFjvkxZ9i+xKH//khElVTmXvFb8
9JD6I00VfHS+Fostio0IknE6xnMuIGYXmsr89P3I5s76DbT9qMhRze3M5F6Ujp1ZQWHQ4dXIqCV/
rUkPBZVaq75aGwfAoU35HRenXjthkFjqRr+nCfpAjAGWSE//IBT4RIhqEcgeNzuIqWmj/dCDKtOR
wXBAzY1HlzebH80zQg2IU1lQa7bT9W7nGe74wGoKRSbMtj8zku36xMAdQVDA3FKbZ49h32UGeF5T
BYK4XelouzfudusPVbA8fagi74S83hziDA7pjw==</ds:X509Certificate>
            </ds:X509Data>
        </ds:KeyInfo>
    </ds:Signature>
    <saml2p:Status xmlns:saml2p="urn:oasis:names:tc:SAML:2.0:protocol">
        <saml2p:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success" />
    </saml2p:Status>
    <saml2:Assertion ID="id16309785467592232019943677"
                     IssueInstant="2021-09-07T15:20:10.400Z"
                     Version="2.0"
                     xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion"
                     >
        <saml2:Issuer Format="urn:oasis:names:tc:SAML:2.0:nameid-format:entity"
                      xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion"
                      >http://www.okta.com/exk1dw29dzLcYLmvg5d7</saml2:Issuer>
        <ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
            <ds:SignedInfo>
                <ds:CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
                <ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256" />
                <ds:Reference URI="#id16309785467592232019943677">
                    <ds:Transforms>
                        <ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature" />
                        <ds:Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
                    </ds:Transforms>
                    <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256" />
                    <ds:DigestValue>NKSkmSe/Ebgtp6+zi/EvvmszRVRMHATsoL/v7z5MBko=</ds:DigestValue>
                </ds:Reference>
            </ds:SignedInfo>
            <ds:SignatureValue>pp9VDci1WNAFYmBfbqqp7ieAbSIW/jcVaDySY5buXDfm60nvVIHVS9ZAitpBJsepHZFvKGsEdbFFSSbczl0ZPXa7Iz4fOUPFH0hOUJSxjdLTP3ooQTSOLGkdE14nw9WXvZh4XjwRZwsHNFsDwxdgnAtvpkeUmPwYswi4BaOkn9pC4yNi8qASzGAShulva6mvW6ErHpJQ8v5Yn4AbKSaFAOywyUKN2pXSFUDM/Flis9dgL+SAytgXa4B3c8loZdsfNKezkeCfCBBNwJgH/w2n6kRIbOshrS4dw8NW7jbYKbQVwbp5wcDYntncsR0/y+lQEZSqhEJp/QC9b5+Ds/gmcQ==</ds:SignatureValue>
            <ds:KeyInfo>
                <ds:X509Data>
                    <ds:X509Certificate>MIIDqDCCApCgAwIBAgIGAXegb/o5MA0GCSqGSIb3DQEBCwUAMIGUMQswCQYDVQQGEwJVUzETMBEG
A1UECAwKQ2FsaWZvcm5pYTEWMBQGA1UEBwwNU2FuIEZyYW5jaXNjbzENMAsGA1UECgwET2t0YTEU
MBIGA1UECwwLU1NPUHJvdmlkZXIxFTATBgNVBAMMDGRldi03NjczODc5NjEcMBoGCSqGSIb3DQEJ
ARYNaW5mb0Bva3RhLmNvbTAeFw0yMTAyMTQxMjA2MDlaFw0zMTAyMTQxMjA3MDlaMIGUMQswCQYD
VQQGEwJVUzETMBEGA1UECAwKQ2FsaWZvcm5pYTEWMBQGA1UEBwwNU2FuIEZyYW5jaXNjbzENMAsG
A1UECgwET2t0YTEUMBIGA1UECwwLU1NPUHJvdmlkZXIxFTATBgNVBAMMDGRldi03NjczODc5NjEc
MBoGCSqGSIb3DQEJARYNaW5mb0Bva3RhLmNvbTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoC
ggEBAMCpSyeOib6kOTMr3cXwZYVyn39cFohSnF3KaXZrd0MU4mDN5VrDl0oXkvTprPM2H9pL+THa
N3smJUEHhiVd7L6ofePCM1njxfpdagAQbGAJAUUyGIQHRY4qoEBEBA3H9TtSi5HY95TwhwfwC3LW
yhmOKBYJLg5YT2CAziKSHopGSueQiy+mYZSSxM6gWNelZW90Yo4hbptqqB2xhoumQg/jSCubqN49
hQf1pzsFibTOmwG/Jmhm8Jr53Q1A3tAo5vvbST2OPK3IrsQPdDlurMEyCjD2DmTVWFqQwkm8fiN3
cGCvOJSYGaHmDncXCZMpJ4Hc6pbaE9e2Nbw2avvXac8CAwEAATANBgkqhkiG9w0BAQsFAAOCAQEA
nQrbS5ucC9WYX1C4nMcUbaZMkaIEg64OtJbn3SzzfbGABrfKonFjvkxZ9i+xKH//khElVTmXvFb8
9JD6I00VfHS+Fostio0IknE6xnMuIGYXmsr89P3I5s76DbT9qMhRze3M5F6Ujp1ZQWHQ4dXIqCV/
rUkPBZVaq75aGwfAoU35HRenXjthkFjqRr+nCfpAjAGWSE//IBT4RIhqEcgeNzuIqWmj/dCDKtOR
wXBAzY1HlzebH80zQg2IU1lQa7bT9W7nGe74wGoKRSbMtj8zku36xMAdQVDA3FKbZ49h32UGeF5T
BYK4XelouzfudusPVbA8fagi74S83hziDA7pjw==</ds:X509Certificate>
                </ds:X509Data>
            </ds:KeyInfo>
        </ds:Signature>
        <saml2:Subject xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion">
            <saml2:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified">jupiteruser01@mycompany.org</saml2:NameID>
            <saml2:SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
                <saml2:SubjectConfirmationData InResponseTo="aah6e85079j2cd414h73ej7bih63ih"
                                               NotOnOrAfter="2021-09-07T15:25:10.400Z"
                                               Recipient="http://localhost:8080/saml/SSO"
                                               />
            </saml2:SubjectConfirmation>
        </saml2:Subject>
        <saml2:Conditions NotBefore="2021-09-07T15:15:10.400Z"
                          NotOnOrAfter="2021-09-07T15:25:10.400Z"
                          xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion"
                          >
            <saml2:AudienceRestriction>
                <saml2:Audience>http://localhost:8080/saml/metadata</saml2:Audience>
            </saml2:AudienceRestriction>
        </saml2:Conditions>
        <saml2:AuthnStatement AuthnInstant="2021-09-07T15:20:10.400Z"
                              SessionIndex="aah6e85079j2cd414h73ej7bih63ih"
                              xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion"
                              >
            <saml2:AuthnContext>
                <saml2:AuthnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport</saml2:AuthnContextClassRef>
            </saml2:AuthnContext>
        </saml2:AuthnStatement>
    </saml2:Assertion>
</saml2p:Response>

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
- https://help.sumologic.com/Manage/Security/SAML/01-Set-Up-SAML-for-Single-Sign-On
- https://javarevisited.blogspot.com/2018/02/what-is-securitycontext-and-SecurityContextHolder-Spring-security.html#axzz75mHj6zQX
- https://addons.mozilla.org/en-US/firefox/addon/saml-tracer/
- https://stackoverflow.com/questions/35893311/getting-list-of-groups-user-is-associated-with-in-okta/37018682#37018682
- https://support.okta.com/help/s/article/How-to-define-and-configure-a-custom-SAML-attribute-statement?language=en_US
- https://developer.okta.com/docs/guides/build-sso-integration/saml2/create-your-app/#create-a-saml-integration


