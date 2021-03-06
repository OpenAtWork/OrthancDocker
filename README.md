# Orthanc for Docker
[Docker Hub](https://www.docker.com/) repository to build [Orthanc](http://www.orthanc-server.com/) and its official plugins. Orthanc is a lightweight, RESTful Vendor Neutral Archive for medical imaging.

## Usage, with plugins disabled

The following command will run the core of Orthanc:

```
# sudo docker run -p 4242:4242 -p 8042:8042 --rm jodogne/orthanc
```

You can also force the [version of Orthanc](https://registry.hub.docker.com/u/jodogne/orthanc/tags/manage/) to be run:

```
# sudo docker run -p 4242:4242 -p 8042:8042 --rm jodogne/orthanc:0.8.6
```

Once Orthanc is running, use Mozilla Firefox at URL [http://localhost:8042/](http://orthanc:orthanc@localhost:8042/app/explorer.html) to interact with Orthanc. The default username is `orthanc` and its password is `orthanc`.

For security reasons, you should protect your instance of Orthanc by changing this default user, in the `RegisteredUsers` configuration option. You can use a custom [configuration file](https://code.google.com/p/orthanc/wiki/OrthancConfiguration) for Orthanc as follows:

```
# sudo docker run --rm --entrypoint=cat jodogne/orthanc /etc/orthanc/orthanc.json > /tmp/orthanc.json
  => Edit /tmp/orthanc.json
  => Modify the RegisteredUsers section
# sudo docker run -p 4242:4242 -p 8042:8042 --rm -v /tmp/orthanc.json:/etc/orthanc/orthanc.json:ro jodogne/orthanc
```

## Usage, with plugins enabled

The following command will run Orthanc, together with its [Web viewer](https://code.google.com/p/orthanc-webviewer/), its [PostgreSQL support](https://code.google.com/p/orthanc-postgresql/) and its [DICOMweb implementation](https://bitbucket.org/sjodogne/orthanc-dicomweb/):

```
# sudo docker run -p 4242:4242 -p 8042:8042 --rm jodogne/orthanc-plugins
```

## PostgreSQL and Orthanc inside Docker

It is possible to run both Orthanc and PostgreSQL inside Docker. First, start the official PostgreSQL container:

```
# sudo docker run --name some-postgres -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=pgpassword --rm postgres
```

Open another shell, and create a database to host the Orthanc database:

```
# sudo docker run -it --link some-postgres:postgres --rm postgres sh -c 'echo "CREATE DATABASE orthanc;" | exec psql -h "$POSTGRES_PORT_5432_TCP_ADDR" -p "$POSTGRES_PORT_5432_TCP_PORT" -U postgres'
```

You will have to type the password (cf. the environment variable `POSTGRES_PASSWORD` above that it set to `pgpassword`). Then, retrieve the IP and the port of the PostgreSQL container, together with the default Orthanc configuration file:

```
# sudo docker inspect --format '{{ .NetworkSettings.IPAddress }}' some-postgres
# sudo docker inspect --format '{{ .NetworkSettings.Ports }}' some-postgres
# sudo docker run --rm --entrypoint=cat jodogne/orthanc-plugins /etc/orthanc/orthanc.json > /tmp/orthanc.json
```

Add the following section to `/tmp/orthanc.json` (adapting the values `Host` and `Port` to what `docker inspect` said above):

```json
  "PostgreSQL" : {
    "EnableIndex" : true,
    "EnableStorage" : true,
    "Host" : "172.17.0.38",
    "Port" : 5432,
    "Database" : "orthanc",
    "Username" : "postgres",
    "Password" : "pgpassword"
  }
```

Finally, you can start Orthanc:

```
# sudo docker run -p 4242:4242 -p 8042:8042 --rm -v /tmp/orthanc.json:/etc/orthanc/orthanc.json:ro jodogne/orthanc-plugins
```
