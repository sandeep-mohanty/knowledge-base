# How Disney Hotstar (now JioHotstar) Scaled Its Infra for 60 Million Concurrent Users

_Disclaimer: The details in this post have been derived from the details shared online by the Disney+ Hotstar (now JioHotstar) Engineering Team. All credit for the technical details goes to the Disney+ Hotstar (now JioHotstar) Engineering Team. The links to the original articles and sources are present in the references section at the end of the post. We’ve attempted to analyze the details and provide our input about them. If you find any inaccuracies or omissions, please leave a comment, and we will do our best to fix them._  

In 2023, Disney+ Hotstar faced one of the most ambitious engineering challenges in the history of online streaming. The goal was to support more than 50 to 60 million concurrent live streams during the Asia Cup and Cricket World Cup. These are events that attract some of the largest online audiences in the world. For perspective, before this, Hotstar had handled about 25 million concurrent users on two self-managed Kubernetes clusters.

To make things even more challenging, the company introduced a “Free on Mobile” initiative, which allowed millions of users to stream live matches without a subscription. This move significantly expanded the expected load on the platform, creating the need to rethink its infrastructure completely.

Hotstar’s engineers knew that simply adding more servers would not be enough. The platform’s architecture needed to evolve to handle higher traffic while maintaining reliability, speed, and efficiency. This led to the migration to a new “X architecture,” a server-driven design that emphasized flexibility, scalability, and cost-effectiveness at a global scale.

The journey that followed involved a series of deep technical overhauls. From redesigning the network and API gateways to migrating to managed Kubernetes (EKS) and introducing an innovative concept called “Data Center Abstraction,” Hotstar’s engineering teams tackled multiple layers of complexity. Each step focused on ensuring that millions of cricket fans could enjoy uninterrupted live streams, no matter how many joined at once.

In this article, we look at how the Disney+ Hotstar engineering team achieved that scale and the challenges they faced.

![](https://substackcdn.com/image/fetch/$s_!rUcp!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fd43e8d32-7140-49cd-8d92-f9bb6199e613_3198x3122.png)

Disney+ Hotstar serves its users across multiple platforms such as mobile apps (Android and iOS), web browsers, and smart TVs.

No matter which device a viewer uses, every request follows a structured path through the system.

-   When a user opens the app or starts a video, their request first goes to an external API gateway, which is managed through Content Delivery Networks (CDNs).
    
-   The CDN layer performs initial checks for security, filters unwanted traffic, and routes the request to the internal API gateway.
    
-   This internal gateway is protected by a fleet of Application Load Balancers (ALBs), which distribute incoming traffic across multiple backend services.
    
-   These backend services handle specific features such as video playback, scorecards, chat, or user profiles, and store or retrieve data from databases, which can be either managed (cloud-based) or self-hosted systems.
    

Each of these layers (from CDN to database) must be fine-tuned and scaled correctly. If even one layer becomes overloaded, it can slow down or interrupt the streaming experience for millions of viewers.

At a large scale, the CDN nodes were not just caching content like images or video segments. They were also acting as API gateways, responsible for tasks such as verifying security tokens, applying rate limits, and processing each incoming request. These extra responsibilities put a significant strain on their computing resources. The system began to hit limits on how many requests it could process per second.

To make matters more complex, Hotstar was migrating to a new server-driven architecture, meaning the way requests flowed and data was fetched had changed. This made it difficult to predict exactly how much traffic the CDN layer would face during a peak event.

To get a clear picture, engineers analyzed traffic data from earlier tournaments in early 2023. They identified the top ten APIs that generated the most load during live streams. What they found was that not all API requests were equal. Some could be cached and reused, while others had to be freshly computed each time.

This insight led to one of the most important optimizations: separating cacheable APIs from non-cacheable ones.

-   Cacheable APIs included data that did not change every second, such as live scorecards, match summaries, or key highlights. These could safely be stored and reused for a short period.
    
-   Non-cacheable APIs handled personalized or time-sensitive data, such as user sessions or recommendations, which had to be processed freshly for each request.
    

By splitting these two categories, the team could optimize how requests were handled. The team created a new CDN domain dedicated to serving cacheable APIs with lighter security rules and faster routing. This reduced unnecessary checks and freed up computing capacity on the edge servers. The result was a much higher throughput at the gateway level, meaning more users could be served simultaneously without adding more infrastructure.

![](https://substackcdn.com/image/fetch/$s_!eP-_!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F2b8dc648-5ac2-42bc-9e20-e60295ae7946_3060x1920.png)

Hotstar also looked at how frequently different features refreshed their data during live matches. For instance, the scorecard or “watch more” suggestions did not need to update every second. By slightly reducing the refresh rate for such features, they cut down the total network traffic without affecting the viewer experience.

Finally, engineers simplified complex security and routing configurations at the CDN layer. Each extra rule increases processing time, and by removing unnecessary ones, the platform was able to save additional compute resources.

Once the gateway layer was optimized, the team turned its attention to the deeper layers of the infrastructure that handled network traffic and computation.

Two key parts of the system required significant tuning: the NAT gateways that managed external network connections, and the Kubernetes worker nodes that hosted application pods.

![](https://substackcdn.com/image/fetch/$s_!HVMn!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F260fe3eb-66e7-467b-a1eb-9125bde614dc_2508x1394.png)
Every cloud-based application depends on network gateways to handle outgoing traffic.

In Hotstar’s setup, NAT (Network Address Translation) Gateways were responsible for managing traffic from the internal Kubernetes clusters to the outside world. These gateways act as translators, allowing private resources in a Virtual Private Cloud (VPC) to connect to the internet securely.

During pre-event testing, engineers collected detailed data using VPC Flow Logs and found a major imbalance: one Kubernetes cluster was using 50 percent of the total NAT bandwidth even though the system was running at only 10 percent of the expected peak load. This meant that if traffic increased five times during the live matches, the gateways would have become a serious bottleneck.

On deeper investigation, the team discovered that several services inside the same cluster were generating unusually high levels of external traffic. Since all traffic from an Availability Zone (AZ) was being routed through a single NAT Gateway, that gateway was being overloaded.

To fix this, engineers changed the architecture from one NAT Gateway per AZ to one per subnet. In simpler terms, instead of a few large gateways serving everything, they deployed multiple smaller ones distributed across subnets. This allowed network load to spread more evenly.

The next challenge appeared at the Kubernetes worker node level. These nodes are the machines that actually run the containers for different services. Each node has limits on how much CPU, memory, and network bandwidth it can handle.

The team discovered that bandwidth-heavy services, particularly the internal API Gateway, were consuming 8 to 9 gigabits per second on individual nodes. In some cases, multiple gateway pods were running on the same node, creating contention for network resources. This could lead to unpredictable performance during peak streaming hours.

The solution was twofold:

-   First, the team switched to high-throughput nodes capable of handling at least 10 Gbps of network traffic.
    
-   Second, they used topology spread constraints, a Kubernetes feature that controls how pods are distributed across nodes. They configured it so that only one gateway pod could run on each node.
    

This ensured that no single node was overloaded and that network usage remained balanced across the cluster. As a result, even during the highest traffic peaks, each node operated efficiently, maintaining a steady 2 to 3 Gbps of throughput per node.

Even after the improvements made to networking and node distribution, the team’s earlier setup had a limitation.

The company was running two self-managed Kubernetes clusters, and these clusters could not scale reliably beyond about 25 million concurrent users. Managing the Kubernetes control plane (which handles how workloads are scheduled and scaled) had become increasingly complex and fragile at high loads.

To overcome this, the engineering team decided to migrate to Amazon Elastic Kubernetes Service (EKS), a managed Kubernetes offering from AWS.

This change meant that AWS would handle the most sensitive and failure-prone part of the system (which is the control plane) while the team could focus on managing the workloads, configurations, and optimizations of the data plane.

Once the migration was complete, the team conducted extensive benchmarking tests to verify the stability and scalability of the new setup. The EKS clusters performed very well during tests that simulated more than 400 worker nodes being scheduled and scaled simultaneously. The control plane remained responsive and stable during most of the tests.

However, at scales beyond 400 nodes, engineers observed API server throttling. In simpler terms, the Kubernetes API server, which coordinates all communication within the cluster, began slowing down and temporarily limiting the rate at which new nodes and pods could be created. This did not cause downtime but introduced small delays in how quickly new capacity was added during heavy scaling.

To fix this, the team optimized the scaling configuration. Instead of scaling hundreds of nodes at once, they adopted a stepwise scaling approach. The automation system was configured to add 100 to 300 nodes per step, allowing the control plane to keep up without triggering throttling.

After migrating to Amazon EKS and stabilizing the control plane, the team had achieved reliable scalability for around 25–30 million concurrent users.

However, as the 2023 Cricket World Cup approached, it became clear that the existing setup still would not be enough to handle the projected 50 million-plus users. The infrastructure was technically stronger but still complex to operate, difficult to extend, and costly to maintain at scale.

Hotstar managed its workloads on two large, self-managed Kubernetes clusters built using KOPS. Across these clusters, there were more than 800 microservices, each responsible for different features such as video playback, personalization, chat, analytics, and more.

Every microservice had its own AWS Application Load Balancer (ALB), which used NodePort services to route traffic to pods. In practice, when a user request arrived, the flow looked like this:  

Client → CDN (external API gateway) → ALB → NodePort → kube-proxy → Pod

See the diagram below:
![](https://substackcdn.com/image/fetch/$s_!-52F!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F1da6d62a-7a97-453d-afd0-428435e14f2d_3198x3878.png)

Although this architecture worked, it had several built-in constraints that became increasingly problematic as the platform grew.

-   **Port Exhaustion:** Kubernetes’ NodePort service type exposes each service on a specific port within a fixed range (typically 30000 to 32767). With more than 800 services deployed, Hotstar was fast running out of available ports. This meant new services or replicas could not be easily added without changing network configurations.
    
-   **Hardware and Kubernetes Version Constraints:** The KOPS clusters were running on an older Kubernetes version (v1.17) and using previous-generation EC2 instances. These versions did not support modern instance families such as Graviton, C5i, or C6i, which offer significantly better performance and efficiency. Additionally, because of the older version, the platform could not take advantage of newer scaling tools like Karpenter, which automates node provisioning and helps optimize costs by shutting down underused instances quickly
    
-   **IP Address Exhaustion:** Each deployed service used multiple IP addresses — one for the pod, one for the service, and additional ones for the load balancer. As the number of services increased, Hotstar’s VPC subnets began running out of IP addresses, creating scaling bottlenecks. Adding new nodes or services often meant cleaning up existing ones first, which slowed down development and deployments.
    
-   **Operational Overhead:** Every time a major cricket tournament or live event was about to begin, the operations team had to manually pre-warm hundreds of load balancers to ensure they could handle sudden spikes in traffic. This was a time-consuming, error-prone process that required coordination across multiple teams.
    
-   **Cost Inefficiency:** Finally, the older cluster autoscaler used in the legacy setup was not fast enough to consolidate or release nodes efficiently.
    

To overcome the limitations of the old setup, Disney+ Hotstar introduced a new architectural model known as Datacenter Abstraction. In this model, a “data center” does not refer to a physical building but a logical grouping of multiple Kubernetes clusters within a specific region. Together, these clusters behave like a single large compute unit for deployment and operations.

Each application team is given a single logical namespace within a data center. This means teams no longer need to worry about which cluster their application is running on. Deployments become cluster-agnostic, and traffic routing is automatically handled by the platform.

See the diagram below:

![](https://substackcdn.com/image/fetch/$s_!XbdH!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Feae8aa58-e398-46f0-842f-724aa3c0c236_3198x3878.png)

This abstraction made the entire infrastructure much simpler to manage. Instead of dealing with individual clusters, teams could now operate at the data center level, which brought several major benefits:

-   **Simplified failover and recovery:** If one cluster faced an issue, workloads could shift to another cluster without changing configurations.
    
-   **Uniform scaling and observability:** Resources across clusters could be managed and monitored as one system.
    
-   **Centralized routing and security:** Rate limiting, authentication, and routing rules could all be handled from a common platform layer.
    
-   **Reduced management overhead:** Engineering teams could focus on applications instead of constantly maintaining infrastructure details.
    

Some key architectural innovations the team made are as follows:

At the core of Datacenter Abstraction was a new central proxy layer, built using Envoy, a modern proxy and load balancing technology. This layer acted as the single point of control for all internal traffic routing.

Before this change, each service had its own Application Load Balancer (ALB), meaning more than 200 load balancers had to be managed and scaled separately. The new Envoy-based gateway replaced all of them with a single, shared fleet of proxy servers.

This gateway handled several critical functions:

-   **Traffic routing:** Directing requests to the right service, regardless of which cluster it was running on.
    
-   **Authentication and rate limiting:** Ensuring that all internal and external requests were secure and properly controlled.
    
-   **Load shedding and service discovery:** Managing temporary overloads and finding the correct service endpoints automatically.
    

By placing this gateway layer within each cluster and letting it handle routing centrally, the complexity was hidden from developers. Application teams no longer needed to know where their services were running; the platform managed everything behind the scenes.

The Datacenter Abstraction model was built entirely on Amazon Elastic Kubernetes Service (EKS), marking a complete move away from self-managed KOPS clusters.

This transition gave Hotstar several important advantages:

-   **Managed control plane:** AWS handled critical control components, reducing maintenance effort and improving reliability.
    
-   **Access to newer EC2 generations:** The team could now use the latest high-performance and cost-efficient instance types, such as Graviton and C6i.
    
-   **Rapid provisioning:** New clusters could be created quickly to meet growing demand.
    
-   **Unified orchestration:** Multiple EKS clusters could now operate together as part of one logical data center, simplifying management across environments.
    

To simplify how services communicate, the team introduced a unified endpoint structure across all environments. Previously, different teams created their own URLs for internal and external access, which led to confusion and configuration errors.

Under the new system, every service followed a clear and consistent pattern:

-   Intra-DC (within the same data center): `<service>.internal.<domain>`
    
-   Inter-DC (between data centers): `<service>.internal.<dc>.<domain>`
    
-   External (public access): `<service>.<public-domain>`
    

This made service discovery much easier and allowed engineers to move a service between clusters without changing its endpoints. It also improved traffic routing and reduced operational friction.

In the earlier architecture, deploying an application required maintaining five or six separate Kubernetes manifest files for different environments, such as staging, testing, and production. This caused duplication and made updates cumbersome.

To solve this, the team introduced a single unified manifest template. Each service now defines its configuration in one base file and applies small overrides only when needed, for example, to adjust memory or CPU limits.

Infrastructure details such as load balancer configurations, DNS endpoints, and security settings were abstracted into the platform itself. Engineers no longer had to manage these manually.

This approach provided several benefits:

-   Reduced duplication of configuration files.
    
-   Faster and safer deployments across environments.
    
-   Consistent standards across all teams.
    

Each manifest includes essential parameters like ports, health checks, resource limits, and logging settings, ensuring every service follows a uniform structure.

One of the biggest technical improvements came from replacing NodePort services with ClusterIP services using the AWS ALB Ingress Controller.

In simple terms, NodePort requires assigning a unique port to each service within a fixed range. This created a hard limit on how many services could be exposed simultaneously. By adopting ClusterIP, services were directly connected to their pod IPs, removing the need for reserved port ranges.

This change made the traffic flow more direct and simplified the overall network configuration. It also allowed the system to scale far beyond the earlier port limitations, paving the way for uninterrupted operation even at massive traffic volumes.

By the time the team completed its transformation into the Datacenter Abstraction model, its infrastructure had evolved into one of the most sophisticated and resilient cloud architectures in the world of streaming. The final stage of this evolution involved moving toward a multi-cluster deployment strategy, where decisions were driven entirely by real-time data and service telemetry.

For every service, engineers analyzed CPU, memory, and bandwidth usage to understand its performance characteristics. This data helped determine which services should run together and which needed isolation. For instance, compute-intensive workloads such as advertising systems and personalization engines were placed in their own dedicated clusters to prevent them from affecting latency-sensitive operations like live video delivery.

Each cluster was carefully designed to host only one P0 (critical) service stack, ensuring that high-priority workloads always had enough headroom to handle unexpected spikes in demand. In total, the production environment was reorganized into six well-balanced EKS clusters, each tuned for different types of workloads and network patterns.

The results of this multi-year effort were remarkable.

Over 200 microservices were migrated into the new Datacenter Abstraction framework, operating through a unified routing and endpoint system. The platform replaced hundreds of individual load balancers with a single centralized Envoy API Gateway, dramatically simplifying traffic management and observability.

When the 2023 Cricket World Cup arrived, the impact of these changes was clear. The system successfully handled over 61 million concurrent users, setting new records for online live streaming without major incidents or service interruptions.

**References:**

-   [Scaling Infrastructure for Millions: From Challenges to Triumphs (Part 1)](https://blog.hotstar.com/scaling-infrastructure-for-millions-from-challenges-to-triumphs-part-1-6099141a99ef)
    
-   [Scaling Infrastructure for Millions: Datacenter Abstraction (Part 2)](https://blog.hotstar.com/scaling-infrastructure-for-millions-datacenter-abstraction-part-2-42b04ef5ed6a)

