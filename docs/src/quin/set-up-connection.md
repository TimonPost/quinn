# Connection Setup

Het opzetten van een quin connectie is het begin van een grote stap naar een betere wereld, echter moet je hier wel wat voor doen.
Het begint bij de `Endpoint` struct, dit is tevens de entry van de library.  
Vanaf uit hier kan je verbindingen opzetten naar andere peers. 
Om een verbinding op te kunnen zetten moet je een aantal configuraties door voeren. 
Configuraties zoals cryptografie en transport instellingen. 
Echter valt dit buiten de scope van dit hoofstuk en wordt daarom hier niet besproken. 
Hoe je een `Endpoint` configureerd kan je in [dit][LINK] hoofstuk vinden.
Omdat we geen configuratie gebruiken zullen de komende voorbeelden dus niet out-of-the box werken.
Als je een `Endpoint` heb opgezet kan je beginnen met het versturen en ontvangen van data, ziet [het volgende hoofdstuk](./data-transfer.md) voor meer informatie.

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

Voor zowel een server als client kunnen we gebruik maken van de `EndpointBuilder`. 
Hiermee kunnen we onze endpoint configuren als een server of client. 
Het is van belang om je endpoint aan een address te koppelen.
Dit doe je doormiddel van `EndpointBuider::bind(address)`. 
Deze functie initializeerd een UDP-socket die door QUIC gebruikt wordt. 
Het is ook mogelijk om vanaf een bestaande socket QUIC te initialiseren. 
Gebruik hier voor `EndpointBuider::with_socket`.

**Client**

In het geval van een client endpoint dien je `connect()` op de endpoint aan te roepen. 
De servernaam staat als het goed is in je certificaat. 

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

In het geval van een server dien je `EndpointBuilder::listen()` aan te roepen met de `ServerConfiguration`. 
Let er op dat dit niets anders is dan het instellen van configuratie en dus geen lusiter logica uitvoert. 

De twee

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