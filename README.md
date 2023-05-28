# Retrieve Keycloak messages in Kafka without SPI.

KC Events is a project for retrieving Keycloak messages in Kafka Message Broker without SPI.
You install and configure Keycloak and Kafka. Then using the connector you connect to the database and fetch every event in the message broker.
All Keycloak events will be recorded in replicated logs, so you can stop and restart each container at any time and the broker gets all the events you missed, ensuring that all events are handled correctly and completely and in the order in which they occurred.

## Requirements

The following software is required to work build it locally:

* [Git](https://git-scm.com) 2.2.1 or later
* [Docker Engine](https://docs.docker.com/engine/install/) or [Docker Desktop](https://docs.docker.com/desktop/) 1.9 or later
* [Curl](https://curl.se/) 7.79.1 or later

See the links above for installation instructions on your platform. You can verify the versions are installed and running:

    $ git --version
    $ curl -V
    $ mvn -version
    $ docker --version

## Usage
### Docker containers

We will use 6 Docker containers which we will deploy using the docker-compose file.

[Zookeeper](https://github.com/apache/zookeeper) - a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services.

[Kafka Cluster](https://kafka.apache.org) - a system that consists of several Brokers, Topics, and Partitions for both. The key objective is to distribute workloads equally among replicas and Partitions.

[Kafdrop](https://github.com/obsidiandynamics/kafdrop): Kafdrop is a web UI for viewing Kafka topics and browsing consumer groups. The tool displays information such as brokers, topics, partitions, consumers, and lets you view messages.

[Postgres](https://www.postgresql.org/) - database for which we want to capture Data Change Events.

[Debezium Connector](https://debezium.io/) - connector captures row-level changes in the schemas of a PostgreSQL database.

[Keycloak](https://www.keycloak.org/) - KC container with custom certificate, for use over `https`. The container is described in [Dockerfile](https://github.com/shvyrev/kc-events/blob/main/docker/Dockerfile).

All containers are described in [docker-compose-kc-events.yaml](https://github.com/shvyrev/kc-events/blob/main/docker/docker-compose-kc-events.yaml).

The file ``sh/run`` is used to deploy the environment and run the containers:

```shell
$ sh/run
```

Once started, the containers are deployed and a connector is added which will pass all events to the message broker.

### Login to KC

After successfully deploying the environment, you need to go to : [https://localhost:8443](https://localhost:8443).

To log in to KC, use admin credentials :
```properties
user : admin
pass : admin
```

### Login Events Settings

To configure messages, go to the `Events` configuration tab: 

**Realm -> Events -> Config**

On the `Login Events Settings` switch on the trigger `Save Events` -> **ON**

In the field `Saved Types` select the types of messages you want to receive in the message broker.

For KC v18.02 it looks like this :

![Login Events Setting](/docs/images/KC.png?raw=true "Login Events Setting for KC v18.02")

At the bottom of the page, click the `Save` button to apply the changes.

### Visualize login events

Now the broker connector is configured and we can see the login events from KC in the message broker.

To do this, just do `Sign out` on the KC admin panel.

To see the messages in the message broker you have to go to the UI interface `Kafdrop` at [http://localhost:9000/topic/kc_events.public.event_entity/messages
](http://localhost:9000/topic/kc_events.public.event_entity/messages).

After that we will see in the UI in topic `kc_events.public.event_entity` the published new messages.

For example like this :

![KafDrop UI](/docs/images/KafDrop.png?raw=true "KafDrop UI")