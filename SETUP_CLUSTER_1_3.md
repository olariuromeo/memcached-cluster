# Basic Memcached Cluster Setup Documentation

## Overview
This document provides instructions for setting up a basic Memcached cluster using Docker, Docker Compose, and mcrouter. The cluster consists of 3 Memcached instances and 1 mcrouter instance, designed to deliver efficient caching for web applications.

### Recommended Server Specifications
For the Basic Cluster configuration, we recommend using a cloud server instance with the following specifications:
- **Instance Type**: **t3.medium** (or equivalent)
- **CPU**: 2 vCPUs
- **RAM**: 4 GB

This setup provides sufficient resources to handle moderate request volumes and ensures efficient caching performance for small applications or development environments.

### Architecture
- **Memcached Instances**: The cluster includes 3 instances of Memcached, each responsible for storing key-value pairs in memory.
- **Mcrouter Instance**: There is 1 instance of mcrouter that acts as a routing layer to distribute requests among the Memcached instances.

## Cluster Configuration

### Docker Compose File
The configuration is defined in the `docker-compose.yaml` file, which is used to create and manage the cluster's services located in the `config` folder. 

### Key Components
- **Memcached Instances**: 3 instances of Memcached are configured, each with unique ports to avoid conflicts. They are responsible for storing and retrieving key-value pairs.
- **Mcrouter Instance**: 1 instance of mcrouter is set up to route requests to the Memcached instances. The mcrouter instance is configured with a command line that specifies the routing policy. 
- **Networking**: All services are connected to a private Docker network to facilitate communication.

## Running the Cluster
To start the cluster, run the following command in the terminal:

```bash
docker compose up -d
```

The `-d` flag runs the containers in detached mode.

## Monitoring
You can monitor the logs of each service using:

```bash
docker compose -f docker-compose.yaml logs -f
```

## Cleanup

To stop and remove all containers and networks, run:

```bash
docker compose down
```

## Data Distribution in the Memcached Cluster

### Overview
In this basic Memcached cluster, data distribution is achieved through sharding and managed by mcrouter. Sharding involves dividing the keyspace among multiple Memcached instances to ensure that no single instance becomes a bottleneck. This improves performance and availability.

### Sharding Mechanism
1. **Hashing Function**: When a key-value pair is stored in Memcached, a hashing function is applied to the key to determine which Memcached instance will hold the value. The hashing function converts the key into a numeric value, which is used to select the appropriate instance based on the total number of instances.
   
   For 3 Memcached instances, the formula to determine which instance to use is:
   ```plaintext
   index = hash(key) % number_of_instances
   ```

   Here, `number_of_instances` is 3, producing an integer between 0 and 2, corresponding to one of the 3 Memcached instances.

2. **Key Distribution**: Each key is routed to one of the 3 Memcached instances based on the hashing result. This ensures an even distribution of data across all instances, facilitating parallel reads and writes.

### Role of mcrouter
- **Routing Requests**: Mcrouter serves as a routing layer between the application and the Memcached instances. When an application requests to set or get a value, it sends the request to mcrouter.
- **Internal Logic**: Mcrouter applies the same hashing logic to determine which Memcached instance should handle the request, simplifying the interaction for the application.
- **Load Balancing**: By managing the distribution of keys, mcrouter helps to balance the load across all Memcached instances. If one instance is overloaded or fails, mcrouter can redirect requests to the other available instances.

### Fault Tolerance and Availability
- **Redundant Copies**: While this configuration does not include data replication, you can implement redundancy by utilizing the multiple instances to store the same key-value pairs. If one instance fails, data can still be accessed from the others.
- **Dynamic Reconfiguration**: If you add or remove Memcached instances, mcrouter can dynamically adjust the routing of requests without requiring changes to the application code.

## Example of Data Flow
1. An application wants to store a value with the key `user:123`.
2. The application sends the request to mcrouter.
3. Mcrouter applies the hashing function to `user:123`, determining that it should be stored in `memcached1`.
4. Mcrouter forwards the request to `memcached1`, which stores the value.
5. When the application requests the value for `user:123`, it sends a request to mcrouter.
6. Mcrouter hashes the key again, finds it belongs to `memcached1`, and retrieves the value.

## Conclusion
In summary, this basic setup provides a straightforward caching solution using Memcached, with mcrouter managing the routing and sharding. This architecture is suitable for applications requiring efficient caching with minimal complexity. 