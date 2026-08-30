1. what is headless service in k8s?
   A headless Service is a Kubernetes Service configured with clusterIP: None. Unlike a normal Service, it doesn't provide a virtual ClusterIP or perform normal Service-level load balancing. Instead, Kubernetes DNS returns the IP addresses of the individual Pods behind the Service. It is commonly used with StatefulSets and distributed applications such as Kafka, Cassandra, or databases where clients need to discover and communicate with individual Pods.

Normal Service:

DNS → ClusterIP → Pod

Headless Service:

DNS → Pod IP(s)
