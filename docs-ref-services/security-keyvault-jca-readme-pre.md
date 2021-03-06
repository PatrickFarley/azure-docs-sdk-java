---
title: Azure KeyVault JCA client library for Java
keywords: Azure, java, SDK, API, azure-security-keyvault-jca, 
author: maggiepint
ms.author: magpint
ms.date: 10/21/2020
ms.topic: article
ms.prod: azure
ms.technology: azure
ms.devlang: java
ms.service: 
---

# JCA Provider for Azure Key Vault

The JCA Provider for Azure Key Vault is a JCA provider for certificates in 
Azure Key Vault. It is built on four principles:
 
1. Must be extremely thin to run within a JVM
1. Must not introduce any library version conflicts with Java app code dependencies
1. Must not introduce any class loader hierarchy conflicts with Java app code dependencies
1. Must be ready for "never trust, always verify and credential-free" Zero Trust environments.

## Testing the version under development

If you want to test the current version under development you will have to
build and install it into your local Maven repository. To do so use the 
following command line:

```
  mvn clean install -DskipTests=true
```

## Server side SSL

If you are looking to integrate the JCA provider to create a SSLServerSocket
see the example below.

```java
    KeyVaultJcaProvider provider = new KeyVaultJcaProvider();
    Security.addProvider(provider);

    KeyStore ks = KeyStore.getInstance("AzureKeyVault");
    KeyVaultLoadStoreParameter parameter = new KeyVaultLoadStoreParameter(
        System.getProperty("azure.keyvault.uri"), 
        System.getProperty("azure.tenant.id"), 
        System.getProperty("azure.client.id"),
        System.getProperty("azure.client.secret"));
    ks.load(parameter);

    KeyManagerFactory kmf = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
    kmf.init(ks, "".toCharArray());

    SSLContext context = SSLContext.getInstance("TLS");
    context.init(kmf.getKeyManagers(), null, null);

    SSLServerSocketFactory factory = (SSLServerSocketFactory) context.getServerSocketFactory();
    SSLServerSocket serverSocket = (SSLServerSocket) factory.createServerSocket(8765);
```

Note if you want to use Azure managed identity, you should set the value
of `azure.keyvault.uri`, and the rest of the parameters would be `null`.

## Client side SSL

If you are looking to integrate the JCA provider for client side socket 
connections, see the Apache HTTP client example below.

```java
    KeyVaultJcaProvider provider = new KeyVaultJcaProvider();
    Security.addProvider(provider);

    KeyStore ks = KeyStore.getInstance("AzureKeyVault");
    KeyVaultLoadStoreParameter parameter = new KeyVaultLoadStoreParameter(
            System.getProperty("azure.keyvault.uri"), 
            System.getProperty("azure.tenant.id"), 
            System.getProperty("azure.client.id"),
            System.getProperty("azure.client.secret"));
    ks.load(parameter);

    SSLContext sslContext = SSLContexts
            .custom()
            .loadTrustMaterial(ks, new TrustSelfSignedStrategy())
            .build();

    SSLConnectionSocketFactory sslSocketFactory = SSLConnectionSocketFactoryBuilder
            .create()
            .setSslContext(sslContext)
            .setHostnameVerifier((hostname, session) -> {
                return true;
            })
            .build();

    PoolingHttpClientConnectionManager cm = PoolingHttpClientConnectionManagerBuilder
            .create()
            .setSSLSocketFactory(sslSocketFactory)
            .build();

    String result = null;

    try ( CloseableHttpClient client = HttpClients.custom().setConnectionManager(cm).build()) {
        HttpGet httpGet = new HttpGet("https://localhost:8766");
        HttpClientResponseHandler<String> responseHandler = (ClassicHttpResponse response) -> {
            int status = response.getCode();
            String result1 = "Not success";
            if (status == 204) {
                result1 = "Success";
            }
            return result1;
        };
        result = client.execute(httpGet, responseHandler);
    } catch (IOException ioe) {
        ioe.printStackTrace();
    }
```

Note if you want to use Azure managed identity, you should set the value
of `azure.keyvault.uri`, and the rest of the parameters would be `null`.

## Spring Boot

For Spring Boot applications see our [Spring Boot starter]<!--(../../spring/azure-spring-boot-starter-keyvault-certificates/README.md)-->.

## Reference

1. [Java Cryptography Architecture (JCA) Reference Guide](https://docs.oracle.com/javase/8/docs/technotes/guides/security/crypto/CryptoSpec.html)

# Azure KeyVault JCA client library for Java - Version 1.0.0-beta.1 


# Getting started

# Key concepts

# Examples

# Troubleshooting

# Next steps

# Contributing

