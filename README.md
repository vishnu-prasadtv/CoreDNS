# CoreDNS
MyLearning

## What is CoreDNS? 

  1. CoreDNS is a DNS server. It is written in Go.
  2. Widely used as the default DNS provider in Kubernetes clusters, created as a replacement for kube-dns in K8s.
  3. CireDNS is highly flexiblae and customisable as each plugins used in CoreDNS provides specific features, like forwarding, caching, or health checking.
  4. Built to handle significant traffic with minimal resource usage, CoreDNS provides rapid DNS responses suitable for dynamic and high-traffic environments.
  5. CoreDNS configuration is done via a simple configuration file (Corefile), allowing flexible setup and management of DNS zones, upstream resolvers, and specific plugins.
   
 ## Installation:
  As Docker cotainer:   
    - You can find the [public Docker hub](https://hub.docker.com/r/coredns/coredns) for the CoreDNS organization. This Docker image is basically scratch + CoreDNS + TLS certificates (for DoT, DoH, and gRPC).
      
 ## Plugins
   - Plugins can be stand-alone or work together to perform a “DNS function”.
   - There are currently about 30 plugins included in the default CoreDNS install.
   - There are also a whole bunch of [external plugins](https://coredns.io/explugins/) that you can compile into CoreDNS to extend its functionality.

 ## Architecture
   As mentioned before CoreDNS is used as DNS provider in Kubernetes clusters, Let us understand how domain name resolution works in Kubernetes

   - Within Kubernetes, when a Pod needs to access a Service in the same Namespace, all it needs to do is execute：
          `# curl aa-svc`

   - But what if the target is in a different namespace? In such cases, you need to include the domain as follows：
          `# curl aa-svc.domain`
       
Reference: https://weng-albert.medium.com/coredns-basic-troubleshooting-resolving-common-issues-en-07c18cd7f8fe

   
