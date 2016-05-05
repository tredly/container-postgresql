## PostgreSQL Container for Tredly

Version 1.0.1 May 5 2016

## Installation

Requires Tredly 0.10.0 <https://github.com/vuid-com/tredly-host> or later

### Modifying container options

By default, the container name is "postgres". Change this by changing containerName in the Tredlyfile prior to building this container.

Many other options can be changed in the Tredlyfile


### Configuring PostgreSQL

postgresql.conf is configured with some example configurations. Please change them for your environment.
pg_hba.conf is configured with some example configurations. You should change these to suit your environment.
recovery.done exists as an example for Master/Slave configuration of PostgreSQL

You will need to create your own SSL keys for Postgres.

Because the PostgreSQL Server communicates via the public internet we need to have a robust way of clients communicating with it.

We need to consider both encrypting the data streams between client and server as well as making sure only approved clients can connect. This is where SSL comes in.

1. Create folder to hold the Server and Client SSL keys. By creating a folder it makes it easier to copy them to the slave servers

```
    mkdir /usr/local/pgsql/ssl
    mkdir /usr/local/pgsql/ssl/clients
```

2. Create the SSL keys for the server

```
    cd /usr/local/pgsql/ssl
    openssl genrsa -des3 -out server.key 4096
    openssl rsa -in server.key -out server.key
    chmod 400 server.key
    chown pgsql:pgsql server.key
```

3. Create a self signed Server Certificate

```
    openssl req -new -key server.key -days 3650 -out server.crt -x509 -subj '/C=AU/ST=QLD/L=Brisbane/O=vuid.com/CN=vuid.com/emailAddress=system@vuid.com'
    cp server.crt root.crt
```

We do not need to do anything further than the above.

The below is IF we want to configure client keys. We do not do this for anything other than replication.

1. Create the client keys

```
    cd /usr/local/pgsql/data/ssl/
    openssl genrsa -des3 -out clients/replication.key 2048
    openssl rsa -in clients/replication.key -out clients/replication.key
```

2. Create client certificate. Note that `Common Name` must be the same as the database user

```
    openssl req -new -key clients/replication.key -out clients/replication.csr -subj '/C=AU/ST=QLD/L=Brisbane/O=vuid.com/CN=replication'
    openssl x509 -req -in clients/replication.csr -CA root.crt -CAkey server.key -out clients/replication.crt -days 365 -CAcreateserial
```

3. Copy the keys to the client machine

```
    /usr/local/pgsql/data/ssl/root.crt
    /usr/local/pgsql/data/ssl/clients/client1.crt
    /usr/local/pgsql/data/ssl/clients/client1.key
```

## License

Tredly is released under the [MIT License](http://www.opensource.org/licenses/MIT).
