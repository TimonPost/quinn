# Certificates

In this chapter, we discuss the configuration of the certificates that are **required** for a working Quinn connection. 

A [Certificate Authority (CA)][ca] is an entity that issues digital [certificates][certificate]. 
These digital certificates certify ownership of a public key associated with, for example, a host, server, client, or document.
Digital certificates ensure that users can be confident that the content actually comes from a reliable, secure source.

By default, Quinn clients validate the cryptographic identity of the servers they connect to. 
This prevents an attacker from intercepting messages.
While it is great that quinn offers security by default it requires additional configuration.
This additional configuration will be the subject of this chapter. 

## Insecure Connection

A certificate is not practical for cases such as: peer-to-peer, trust-on-first-use, deliberately insecure applications, or when the servers are not identified by the domain name. Custom certificate validation logic can be implemented when the `dangerous_configuration` function of [rustls][rust-ls] is enabled. 
Once done, the certificate verifier can manually override in the `quinn::ClientConfig`.

Start with adding a [rustls][rust-ls] dependency with the `dangerous_configuration` feature flag to your toml file.

```toml
quinn = "*"
rustls = { version = "*", features = ["dangerous_configuration", "quic"] }
``` 

Then, you can skip the certificate validation on the client by implementing `rustls::ServerCertVerifier` and let it always return 'correct'.

```rust
// Implementation of `ServerCertVerifier` that verifies everything as trustworthy.
struct SkipCertificationVerification;

impl rustls::ServerCertVerifier for SkipCertificationVerification {
    fn verify_server_cert(
        &self,
        _roots: &rustls::RootCertStore,
        _presented_certs: &[rustls::Certificate],
        _dns_name: webpki::DNSNameRef,
        _ocsp_response: &[u8],
    ) -> Result<rustls::ServerCertVerified, rustls::TLSError> {
        Ok(ServerCertVerified::assertion())
    }
}
```

After that, we can configure our `ClientConfig` to use this new `CertificateVerifier`. 

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
 
Finally, if you throw this `ClientConfig` into the `Endpoint::default_client_config()` your endpoint should verify all connections as trustworthy.

## Using Certificates

In this section we look at certifying an endpoint with a real certificate. 
This can be done with a real certificate, but also with a self-identified certificate. 

Let's define two useful functions that can dissect byte certificates and return quinn types.

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

There are two common certificate formats namely: `.pem` and `.der`.
The `.der` certificates are byte-coded, while `.pem` is text-coded.
You can translate one to the other by using tooling such as openssl or even within code self. 
The code translation is shown above. 

### Self Signed

A [self-signed][self-signed] certificate is a security certificate that is unknown not by a CA but by yourself. 
These certificates are easy to create and cost no money. 
However, they do not offer all the security features that certificates from a CA do have. 
You can create a self-signed certificate using [rcgen][rcgen] and write it to a permanent place. 
Note that `generate_simple_self_signed` supports both `.der` and `.pem` formatting.

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

[Let's Encrypt][lets-encrypt] is a CA and distributes certificates for free. 
Its a very well-known CA used by many applications around the world.
We can cover a full lets-encrypt tutorial here but instead I am going to keep it short. 
There is plenty of good documentation out there that can help you further.  

**Generate Certificate**

Let's Encrypt works with [Certbot][certbot], certbort generates the certificate for you.
Often a certificate is generated to secure a web server. Because we generate a certificate for a protocol, the configuration process will be slightly different than normal. We assume that you do not have a web server. 
Select on the certbot website that you do not have a web server and follow the given installation instructions.

If certbot is installed, execute `certbot certonly --standalone`, this command will run a web server in the background during the process.
Cerbot asks for your data, after entering it two `.pem` files are generated, namely `cert.pem` and `privkey.pem`. Copy them to your project because we will reference them in code. 
 
```rust
// Read from certificate and key from directory. 
let (cert, key) = fs::read(&"./cert.pem").and_then(|x| Ok((x, fs::read(&"./privkey.pem")?)))?;
// Parse bytes to type.
parse_pem(cert, key)
```

### Configuring Certificates

Now you generated, or maybe you already had, the certificate, they need to be configured into the client and server. 

**Configure Server**

*`certificate` is of type `Certificate` and `key` of type `PrivateKey`.*

```rust
let mut builder = ServerConfigBuilder::default();
builder.certificate(CertificateChain::from_certs(vec![certificate]), key)?;
```

This is the only thing you need to do for your sever to be secured. 

**Configure Client**

*`certificate` is of type `Certificate`

```rust
let mut builder = ClientConfigBuilder::default();
builder.add_certificate_authority(certificate)?;    
```

This is the only thing you need to do for your client to be secured.

[Nextup](set-up-connection.md), we will investigate how to use this setup an connection with this certificate. 

[certbot]: https://certbot.eff.org/instructions
[lets-encrypt]: https://letsencrypt.org/getting-started/
[rust-ls]: https://github.com/ctz/rustls
[rcgen]: https://github.com/est31/rcgen
[self-signed]: https://en.wikipedia.org/wiki/Self-signed_certificate#:~:text=In%20cryptography%20and%20computer%20security,a%20CA%20aim%20to%20provide.
[certificate]: https://en.wikipedia.org/wiki/Public_key_certificate
[ca]: https://en.wikipedia.org/wiki/Certificate_authority