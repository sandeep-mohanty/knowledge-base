# Importing p7b File to Java Keystore Using Keytool in Java | Baeldung

## 1\. Introduction

When working with production applications in Java, we often find ourselves configuring HTTPS, enabling secure outbound communication, or managing trust stores. **[Keytool](/keytool-intro) is the standard tool for working with the required keys and certificates and thus managing Java [keystores](/java-keystore).**

In this tutorial, we’ll focus on a specific task: importing a _p7b_ certificate file into a Java keystore using keytool.

## 2\. Understanding the P7B (PKCS#7) Format

Before we dive into the steps to import a P7B file, it’s worth reviewing and understanding the [PKCS#7 format](https://www.rfc-editor.org/rfc/rfc2315.html).

**In cryptography, PKCS#7 is a standard syntax for storing cryptographically signed or encrypted data**.

One common use for this format is to store SSL certificates. Often, certificate bundles are stored and shared as a _.p7b_ file format.

### 2.1. Certificate Bundle

When creating [self signed](/spring-boot-https-self-signed-certificate) certificates for development use, it’s usually sufficient to have a private key and a self-signed certificate containing the public key. However, when working with production applications that are expected to be available over the internet, we typically need our certificates to be validated by a known Certificate Authority (CA).

**Browsers validate a site by validating the site’s certificate.** Each site presents its own certificate along with information to a chain of intermediate certificates, eventually leading to the root certificate of a CA. Since the browsers trust the CA, any site presenting a validated certificate with a clear chain of certificates leading to a root CA certificate is considered valid.

Once a certificate is validated by a CA, they typically provide a certificate chain consisting of its own root certificate and zero or more intermediate certificates. This certificate chain is provided as a bundle of certificates, sometimes as a _.p7b_ file.

### 2.2. The P7B Format

**A P7B file is a PKCS#7 container that can store one or more certificates.** It may be encoded either in the binary DER format or in the Base64 PEM format.

When working with Java servers (such as Tomcat), we need to ensure that all the certificates in the bundle are imported into our keystore to prove the validity of our certificate.

Browsers on all modern devices recognise certificates validated by a CA. **During the process of certificate validation, a CA will typically share the signed public certificate for our site along with a certificate chain, aka certificate bundle**.

Sometimes, the certificate bundle comes as a file with a _.p7b_ extension. We note that depending on the encoding, the contents may not be easily readable in a plain text editor. Therefore, we need a few tools to work with these certificates and import them into our keystore.

## 3\. Preparing a P7B file

Now that we’ve understood the P7B format, let’s dive into the process.

**Before we can import a certificate bundle, we need a file containing the certificate bundle**. Since we don’t have a _.p7b_ provided by a CA, we’ll use the public certificate from [Baeldung.com](/).

First, we’ll download the chain and convert it into the P7B file format.

### 3.1. Downloading the Certificate Bundle

Most modern browsers allow exporting of certificates to PKCS#7 format encoded files. However, we’ll do a full command-line interface version using the [_openssl_](/linux/openssl-command-examples) command on a Linux terminal:

```
$ openssl s_client -connect www.baeldung.com:443 -showcerts </dev/null
```

This command prints out several certificates along with some other information about each certificate. Each certificate can be identified easily because of the beginning and ending markers:

```
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
```

We can now copy each of those certificates (including the markers) and store them as a separate _.pem_ file. So now we have three files _site.pem, intermediate.pem and root.pem_.

Now let’s use them to create a _.p7b_ bundle:

```
$ openssl crl2pkcs7 -nocrl -certfile site.pem -certfile intermediate.pem \
-certfile root.pem -out site-chain.p7b
```

Finally, we can verify that all certificates of interest are present in the bundle:

```
$ openssl pkcs7 -print_certs -in site-chain.p7b -noout
```

We see three certificates printed clearly as expected.

### 3.2. The Problem Importing P7B file

Now that we have a valid certificate bundle in the _.p7b_ file format, let’s check it with _keytool_:

```
$ keytool -printcert -file site-chain.p7b
```

This command prints information about the three certificates clearly.

Let’s attempt to import this as-is:

```
$ keytool -importcert -file site-chain.p7b -keystore test-keystore.jks -alias site
```

The output presents an error:

```
keytool error: java.lang.Exception: Input not an X.509 certificate
```

**The certificate import fails because the _keytool -importcert_ expects the input file to contain exactly one X.509 PEM or DER encoded certificate.**

We observe that the direct import of a certificate bundle using keytool can be problematic. **The solution is to break down the bundle into its individual certificates and then import those certificates one by one.**

### 4.1. Converting to PEM Encoding

To break the bundle into individual PEM files, first, we’ll convert the bundle from the PKCS #7 to a PEM encoded bundle:

```
$ openssl pkcs7 -print_certs -in site-chain.p7b -out site-chain.pem
```

This produces a PEM file containing one or more X.509 certificates:

We’re now ready to use _keytool_ to import the certificates.

### 4.2. Importing the PEM Bundle

**Now that the file is in a format compatible with the _keytool_ import option, we can perform the input**:

```
$ keytool -importcert -file site-chain.pem -keystore test-keystore.jks -alias site
```

This command requires us to set a keystore password and asks us if we trust the certificate. Then the output shows success:

```
Trust this certificate? [no]:  yes
Certificate was added to keystore
```

This imports the site’s certificate under the alias _site_.

**If we need the intermediate and root certificates as well in the same keystore, then we need to import them individually**. From _site-chain.pem_ we can copy each of those certificates (including the begin and end markers) and store them as a separate _.pem_ file. So now we have two more files, _intermediate.pem_ and _root.pem_.

We can use the same command with different input files and aliases to import the root and intermediate certificates as well:

```
$ keytool -importcert -file root.pem -keystore test-keystore.jks -alias root
$ keytool -importcert -file intermediate.pem -keystore test-keystore.jks -alias intermediate
```

### 4.3. Verifying the Import

Now we have all certificates imported, let’s verify them:

```
$ keytool -list -v -keystore test-keystore.jks
```

This command requests the previously set password and then shows us detailed information about all the imported certificates. At the beginning of the output, we see:

```
Keystore type: PKCS12
Keystore provider: SUN

Your keystore contains 3 entries
```

That’s it. Our keystore is now ready to use.

## 5\. Conclusion

In this article, we explored how keytool works with certificate bundles in the PKCS#7 (_.p7b_) format. While these files can contain complete certificate chains, they’re not always directly supported for import.

We learnt that _keytool_ expects X.509 certificates in DER or PEM format, not container formats like PKCS#7. **Converting the _.p7b_ bundle into individual certificates ensures a successful and predictable import process**.