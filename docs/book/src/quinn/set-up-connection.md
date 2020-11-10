# Connection Setup

In het [vorige hoofdstuk](certificate.md) is gekeken hoe we een certificaat kunnen configureren, dit aspect wordt daarom hier achterwege gelaten. 
In dit hoofstuk wordt toegelicht hoe een connectie opgezet wordt.

Een Quinn connectie kan opgezet worden met de `Endpoint` struct, dit is tevens de entry van de library. 

## Example

Laten we beginnen met het defineren van wat constanten. 

```rust
static SERVER_NAME: &str = "localhost";

fn client_addr() -> SocketAddr {
    "127.0.0.1:5000".parse::<SocketAddr>().unwrap()
}

fn server_addr() -> SocketAddr {
    "127.0.0.1:5001".parse::<SocketAddr>().unwrap()
}
```   

Voor zowel een server als client maken we gebruik van de `EndpointBuilder`. 
De `EndpointBuilder` heeft een methode `EndpointBuider::bind(address)` waarmee je een address koppelt aan de endpoint. 
Deze methode initializeerd een UDP-socket die quinn gebruikt.
Daarnaast is het ook mogelijk om vanaf een bestaande socket een quinn endpoint te initialiseren. 
Dit kan met de `EndpointBuider::with_socket` methode.

**Client**

Net als een TCP-client dien je een verbinding met een destination te maken. 
In quinn kan dit met de methode `connect()`. 
De `connect` methode heeft een argument 'servernaam', dit is de naam die in het certificaat staat. 

```rust
async fn client() -> anyhow::Result<()> {
    let mut endpoint_builder = Endpoint::builder();

    // Bind this endpoint to a UDP socket on the given client address.
    let (endpoint, _) = endpoint_builder.bind(&client_addr())?;

    // Connect to the server passing in the server name which is supposed to be in the server certificate.
    let connection: NewConnection = endpoint
        .connect(&server_addr(), SERVER_NAME)?
        .await?;

    // Start transferring, receiving data, see DataTransfer tutorial.

    Ok(())
}
```

**Server**

Net als een TCP Listener dien je naar inkomende connecties te luisteren. 
Voor dat je kan luisteren naar connecties moet je met de `EndpointBuilder` de `Endpoint` als server configureren.  
Let er op dat de configuratie zelf geen luister logica uitvoert, dit kan pas als je `bind()` hebt uitgevoert.  

```rust
async fn server() -> anyhow::Result<()> {
    let mut endpoint_builder = Endpoint::builder();
    // Configure this endpoint as a server by passing in `ServerConfig`.
    endpoint_builder.listen(ServerConfig::default());

    // Bind this endpoint to a UDP socket on the given server address. 
    let (endpoint, mut incoming) = endpoint_builder.bind(&server_addr())?;

    // Start iterating over incoming connections.
    while let Some(conn) = incoming.next().await {
        let mut connection: NewConnection = conn.await?;

        // Save connection somewhere, start transferring, receiving data, see DataTransfer tutorial.
    }

    Ok(())
}
```