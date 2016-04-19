**PostgreSQL Container for Tredly**

For Tredly-Build Version 0.9.0

**Installation**

This container requires Tredly-Host to build and run it, https://github.com/vuid-com/tredly-build

**Modifying container options**

By default, the container name is "postgres". Change this by changing containerName in the Tredlyfile prior to building this container.

Many other options can be changed in the Tredlyfile


**Configuring PostgreSQL**

postgresql.conf is configured with some example configurations. Please change them for your environment.
pg_hba.conf is configured with some example configurations. You should change these to suit your environment.
recovery.done exists as an example for Master/Slave configuration of PostgreSQL

You will need to create your own SSL keys for Postgres.

Because the PostgreSQL Server communicates via the public internet we need to have a robust way of clients communicating with it.

We need to consider both encrypting the data streams between client and server as well as making sure only approved clients can connect. This is where SSL comes in.

    Create folder to hold the Server and Client SSL keys
        By creating a folder it makes it easier to copy them to the slave servers
        mkdir /usr/local/pgsql/ssl
        mkdir /usr/local/pgsql/ssl/clients
    Create the SSL keys for the server
        cd /usr/local/pgsql/ssl
        Generate a 4096 bit long Private Key
            openssl genrsa -des3 -out server.key 4096
            Pick any old passphrase as we need to remove it or Postgres requires it to be entered on startup
        Remove the Private Key passphrase
            openssl rsa -in server.key -out server.key
        Set correct permissions for the Private Key

            chmod 400 server.key
            chown pgsql:pgsql server.key
        Create a self signed Server Certificate
            openssl req -new -key server.key -days 3650 -out server.crt -x509 -subj '/C=AU/ST=QLD/L=Brisbane/O=vuid.com/CN=vuid.com/emailAddress=system@vuid.com'
        Since we are self signing we use the server certificate as the trusted root certificate
            cp server.crt root.crt

We do not need to do anything further than the above. The below is IF we want to configure client keys. We do not do this for anything other than replication.

    Create the client keys
        Naming of client keys is important so we can keep track of them.
        We only currently create a single key for the replication user
        cd /usr/local/pgsql/data/ssl/
        openssl genrsa -des3 -out clients/replication.key 2048
            Pick any only passphrase as we are going to remove it in a second.
        Remove the passphase from key
            openssl rsa -in clients/replication.key -out clients/replication.key
            Your going to need to enter the password set for the client key.
        Create client certificate
            Common Name must be the same as the database user
            openssl req -new -key clients/replication.key -out clients/replication.csr -subj '/C=AU/ST=QLD/L=Brisbane/O=vuid.com/CN=replication'
        openssl x509 -req -in clients/replication.csr -CA root.crt -CAkey server.key -out clients/replication.crt -days 365 -CAcreateserial
    Copy the keys to the client machine
        /usr/local/pgsql/data/ssl/root.crt
        /usr/local/pgsql/data/ssl/clients/client1.crt
        /usr/local/pgsql/data/ssl/clients/client1.key