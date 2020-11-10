# Certificates

A Certificate Authority (CA) is an entity that issues digital certificates. 
These digital certificates certify the ownership of a public key associated with a host, server, client, document, and more. 
Digital certificates help to ensure users can trust that your content is actually from a reliable, safe source.

By default, Quinn clients validate the cryptographic identity of servers they
connect to. This prevents an active, on-path attacker from intercepting
messages, but requires trusting some certificate authority and thus some configuration up-front. 
However, it is possible to use Quinn over an insecure connection or self-sign your certificates. 

## Insecure Connection

For some cases, including peer-to-peer, trust-on-first-use, deliberately
insecure applications, or any case where servers are not identified by domain
name, a certificate isn't practical. Arbitrary certificate validation logic can be
implemented by enabling the `dangerous_configuration` feature of `rustls`. 
In addition you need to override the certificate verifier by hand in the `ClientConfig`.

First, add a `rustls` dependency with the `dangerous_configuration` feature flag to your toml file.

```toml
quinn = "*"
rustls = { version = "*", features = ["dangerous_configuration", "quic"] }
``` 

Then, you can skip the certificate check on the client by implementing `rustls::ServerCertVerifier` and let it always return 'correct'.
After that, we should configure our client to use this certificate verifier. 

```rust
pub fn insecure() -> ClientConfig {
    let mut cfg = quinn::ClientConfigBuilder::default().build();

    // Get a mutable reference to the 'crypto' config in the 'client config'..
    let tls_cfg: &mut rustls::ClientConfig =
        std::sync::Arc::get_mut(&mut cfg.crypto).unwrap();

    // Change the certification verifier.
    // This is only available when compiled with 'dangerous_configuration' feature.
    tls_cfg
        .dangerous()
        .set_certificate_verifier(Arc::new(SkipCertificationVerification));
    cfg
}
```
 
Now, if you throw this `ClientConfig` into the `Endpoint::default_client_config()` your endpoint should verify all connections as trustworthy.

## Using Certificates

In this section we look at how to create a certificate for Quinn. 
First we deal with the self-signed certificate, and after that the real certificates.  

There are two common types of certificate formats. Those are `.pem` and `.der`.
The `.der` certificates are byte encoded while `.pem` are text encoded.  
You can convert on to the other by using tooling such as openssl or also withing code its self as demonstrated below. 

Let's define two useful function that can dissect byte certificates and return nice instances of structures. 

```rust
pub fn parse_der(cert: Vec<u8>, private_key: Vec<u8>) -> anyhow::Result<(quinn::Certificate, quinn::PrivateKey)> {
    let cert = quinn::Certificate::from_der(&cert)?;
    let key = quinn::PrivateKey::from_der(&private_key)?;
    Ok((cert, key))
}

pub fn parse_pem(cert: Vec<u8>, private_key: Vec<u8>) -> anyhow::Result<(quinn::Certificate, quinn::PrivateKey)> {
    // Parse to certificate chain whereafter taking the first certificate in this chain.
    let cert = quinn::CertificateChain::from_pem(&cert)?.iter().next().unwrap().clone();
    let key = quinn::PrivateKey::from_pem(&private_key)?;

    Ok((quinn::Certificate::from(cert), key))
}
```

### Self Signed
Sometimes working with your own certificate authority makes no sense, then you can use self-signed certificates. 
For this purpose [rcgen][rcgen] can be used.
You need to write the certificate to permanent storage for later reuse.
Notice that `generate_simple_self_signed` supports both `.der` and `.pem` formatting.

Let's look at an example:

```rust
pub fn generate_self_signed_cert(cert_path: &str, key_path: &str) -> anyhow::Result<(quinn::Certificate, quinn::PrivateKey)> {
    // Generate dummy certificate.
    let certificate = rcgen::generate_simple_self_signed(vec!["localhost".into()]).unwrap();
    let serialized_key = certificate.serialize_private_key_der();
    let serialized_certificate = certificate.serialize_der().unwrap();

    // Write to files.
    fs::write(&cert_path, &serialized_certificate).context("failed to write certificate")?;
    fs::write(&key_path, &serialized_key).context("failed to write private key")?;

    parse_der(serialized_certificate, serialized_key)
}
```

### Official Certificates

Let’s Encrypt is a CA and gives out certificates for free. 
Its a very well-known CA used for many applications around the world.
We can cover a full lets-encrypt tutorial here but instead I am going to keep it short. 
There is plenty of documentation out there which keeps itself up-to-date. 

**Generate Certificate**

Official certificates can be requested from an organization such as [Let's Encrypt][lets-encrypt].
Let's Encrypt works together with [Certbot][certbot] who will generate the certificate for you.
Because we generate a certificate for a protocol we assume that you are not using a web server.
Select on the certbot website that you do not have a web server and follow the given installation instructions.

If certbot is installed, run `certbot certonly --standalone`, this option assumes that you do not have a web server installed and will therefore run a web server during the process.
After entering your information and domain addresses two `.pem` files, namely: Con `cert.pem` and `privkey.pem` will be generated.
As soon as they are generated, copy them to your project. 
 
```rust
// Read from certificate and key from directory. 
let (cert, key) = fs::read(&"./cert.pem").and_then(|x| Ok((x, fs::read(&"./privkey.pem")?)))?;
// Parse bytes to type.
parse_pem(cert, key)
```

### Configuring Certificates

When you generated the certificate it needs to be configured into the client and server. 
`certificate` is of type `Certificate` and `key` of type `PrivateKey`.

**Configure Server**

```rust
let mut builder = ServerConfigBuilder::default();
builder.certificate(CertificateChain::from_certs(vec![certificate]), key)?;
```

**Configure Client**

```rust
let mut builder = ClientConfigBuilder::default();
builder.add_certificate_authority(certificate)?;    
```

If done well, your endpoint should be encrypted.

[Nextup](set-up-connection.md), we will have a take on how to setup the connection.

[certbot]: https://certbot.eff.org/instructions
[lets-encrypt]: https://letsencrypt.org/getting-started/