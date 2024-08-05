# Distributed Messaging System with RabbitMQ

## Overview

This project demonstrates the development and deployment of a robust distributed messaging system using RabbitMQ, Docker, and GoLang. The system is designed to handle message transmission and reception with high reliability and scalability. Key components include sender and receiver programs written in GoLang, which are containerized using Docker to ensure consistent and portable deployment. Kubernetes is utilized for orchestration to manage scaling, ensure high availability, and handle fault tolerance.

The system architecture is built to support queue mirroring, allowing for message redundancy across multiple RabbitMQ nodes, enhancing system reliability and fault tolerance.

## Features

- **GoLang Programs**: 
  - **Sender Program**: Handles the creation and sending of messages to RabbitMQ queues.
  - **Receiver Program**: Consumes and processes messages from RabbitMQ queues.
- **RabbitMQ**: 
  - Provides a reliable and scalable messaging platform with advanced features like queue mirroring to ensure message availability and fault tolerance.
- **Docker**: 
  - Used for containerizing the GoLang programs and RabbitMQ instances, simplifying deployment and management across different environments.
- **Kubernetes**: 
  - Manages container orchestration, scaling, and automatic recovery, ensuring the system can handle increased loads and recover from failures.

## Setup and Management

### RabbitMQ Basic Queue Mirroring

To set up queue mirroring in RabbitMQ, follow these steps:

1. **Access RabbitMQ Container:**
    ```bash
    sudo docker exec -it rabbit-1 bash
    ```

2. **Set Queue Mirroring Policy:**
    - For standard mirroring:
      ```bash
      rabbitmqctl set_policy ha-fed ".*" '{"federation-upstream-set":"all", "ha-mode":"nodes", "ha-params":["rabbit@rabbit-1","rabbit@rabbit-2","rabbit@rabbit-3"]}' --priority 1 --apply-to queues
      ```
      This command sets a policy that mirrors queues across the specified nodes.
      
    - For automatic synchronization:
      ```bash
      rabbitmqctl set_policy ha-fed ".*" '{"federation-upstream-set":"all", "ha-sync-mode":"automatic", "ha-mode":"nodes", "ha-params":["rabbit@rabbit-1","rabbit@rabbit-2","rabbit@rabbit-3"]}' --priority 1 --apply-to queues
      ```
      This policy ensures that queues are automatically synchronized with all nodes.

### Message Publisher and Consumer

To build and run the message publisher and consumer, use the following commands:

1. **Build and Run Message Publisher:**
    ```bash
    cd application/publisher
    sudo docker build . -t aimvector/rabbitmq-publisher:v1.0.0
    sudo docker run -it --rm --net rabbits -e RABBIT_HOST=rabbit-1 -e RABBIT_PORT=5672 -e RABBIT_USERNAME=guest -e RABBIT_PASSWORD=guest -p 80:80 aimvector/rabbitmq-publisher:v1.0.0
    ```
    This will build the Docker image for the publisher and run it, connecting to the RabbitMQ instance.

2. **Build and Run Message Consumer:**
    ```bash
    sudo docker build . -t aimvector/rabbitmq-consumer:v1.0.0
    sudo docker run -it --rm --net rabbits -e RABBIT_HOST=rabbit-1 -e RABBIT_PORT=5672 -e RABBIT_USERNAME=guest -e RABBIT_PASSWORD=guest aimvector/rabbitmq-consumer:v1.0.0
    ```
    This builds the Docker image for the consumer and runs it, allowing it to consume messages from the RabbitMQ instance.

### Management

For managing RabbitMQ instances, including setting up clusters and retrieving configuration details, follow these steps:

1. **Run RabbitMQ Instance:**
    ```bash
    docker network create rabbits
    docker run -d --rm --net rabbits --hostname rabbit-1 --name rabbit-1 rabbitmq:3.8
    ```
    This command creates a Docker network and runs a RabbitMQ instance in a container.

2. **Retrieve Erlang Cookie:**
    ```bash
    docker exec -it rabbit-1 cat /var/lib/rabbitmq/.erlang.cookie
    ```
    This retrieves the Erlang cookie used for authentication between RabbitMQ nodes.

3. **Clean Up RabbitMQ Instance:**
    ```bash
    docker rm -f rabbit-1
    ```
    This removes the RabbitMQ container instance.

4. **Run RabbitMQ Management Container:**
    ```bash
    docker run -d --rm --net rabbits -p 8080:15672 -e RABBITMQ_ERLANG_COOKIE=DSHEVCXBBETJJVJWTOWT --hostname rabbit-manager --name rabbit-manager rabbitmq:3.8-management
    ```
    This starts a RabbitMQ management container to provide a web-based management interface.

5. **Join RabbitMQ Manager to Cluster:**
    ```bash
    docker exec -it rabbit-manager rabbitmqctl stop_app
    docker exec -it rabbit-manager rabbitmqctl reset
    docker exec -it rabbit-manager rabbitmqctl join_cluster rabbit@rabbit-1
    docker exec -it rabbit-manager rabbitmqctl start_app
    docker exec -it rabbit-manager rabbitmqctl cluster_status
    ```
    These commands stop the RabbitMQ application, reset its state, join it to the cluster, and then start the application and check the cluster status.

## References

- [RabbitMQ High Availability](https://www.rabbitmq.com/ha.html#unsynchronised-mirrors)
- [RabbitMQ Cluster Formation](https://www.rabbitmq.com/cluster-formation.html)
- [RabbitMQ High Availability Modes](https://www.rabbitmq.com/ha.html)

## Acknowledgments

- [RabbitMQ](https://www.rabbitmq.com/)
- [Docker](https://www.docker.com/)
- [Kubernetes](https://kubernetes.io/)
- [GoLang](https://golang.org/)
