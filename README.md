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

  Using Kubeadm: https://coredns.io/2018/05/21/migration-from-kube-dns-to-coredns/#installing-coredns-via-kubeadm  
      
 ## Plugins
   - Plugins can be stand-alone or work together to perform a “DNS function”.
   - There are currently about 30 plugins included in the default CoreDNS install.
   - There are also a whole bunch of [external plugins](https://coredns.io/explugins/) that you can compile into CoreDNS to extend its functionality.

 ## Architecture
   As mentioned before CoreDNS is used as DNS provider in Kubernetes clusters, Let us understand how domain name resolution works in Kubernetes

   - Within Kubernetes, when a Pod needs to access a Service in the same Namespace, all it needs to do is execute：

     ``# curl pod1-svc ``

   - But what if the target is in a different namespace? In such cases, you need to include the domain as follows：

      ``# curl pod1-svc.domain``

   So when the service name of a pod outside namespace has to be resolved, the DNS resolution is required. 
   Generally for DNS resolution, below files are used in Linux:
   
     /etc/hosts.conf
     /etc/hosts 
     /etc/resolv.conf
      
In Kubernetes CoreDNS make use of four DNS policies for resolution, particularly in handling the relationship between internal and external DNS queries:

  1. **Default**: This policy lets kubelet determine which DNS policy to use, with the default being to use the `resolv.conf` content from the Node. 
  
  2. **ClusterFirst**: This policy prioritizes using Kubernetes' internal DNS service (specifically CoreDNS) for DNS resolution within Pods. 
  
  3. **ClusterFirstWithHostNet**: When a Pod uses `hostNetwork: true`, it directly adopts the `resolv.conf` contents from the Node. 
  
  4. **None**: This policy doesn't specify a DNS policy, allowing you to customize DNS configuration using dnsConfig.


## CoreFile

  The Corefile is CoreDNS’s configuration file. It defines:
  
    1. What servers listen on what ports and which protocol.
    2. For which zone each server is authoritative.
    3. Which plugins are loaded in a server.
  
       
  To explain more, let take a look at this “Corefile”:
  
  ```
  ZONE:[PORT] {
      [PLUGIN]...
  }
  ```
  
  **ZONE** - Defines the zone this server. The optional PORT defaults to 53, or the value of the -dns.port flag.
  
  **PLUGIN** - Defines the plugin(s) we want to load. This is optional as well, but a server with no plugins will just return SERVFAIL for all queries. Each plugin can have a number of properties than can have arguments
  
  **Example:**
  
  The ZONE is root zone ., the PLUGIN is chaos. The chaos plugin does not have any properties, but it does take an argument: CoreDNS-001. This text is returned on a CH class query: dig CH txt version.bind @localhost
  ```
  . {
     chaos CoreDNS-001
  }
  ```
  
  **NOTE**:
  If CoreDNS can’t find a Corefile to load, it loads the following builtin one that loads the [whoami plugin](https://coredns.io/plugins/whoami/):
  ```
  . {
      whoami
  }
  ```
  
  **Servers**
  This is the most minimal Corefile:
  
  ``. { }``
  That defines a server to listen on port 53 and make it authoritative for the root zone and everything below. Let’s define another server that is authoritative for . (root zone) and load that:
  
  ```
  . { }
  . { }
  ```
  
  This will make CoreDNS exit with an error:
  ``2017/07/23 20:39:10 cannot serve dns://.:53 - zone already defined for dns://.:53``
  
  Why? Because we already defined a server on the same port for this zone. If we change the port number on the second server and thereby creating another server, it is OK:
  ```
  .    { }
  .:54 { }
  ```
  When defining a new zone, you either create a new server, or add it to an existing one. Here we define one server that handles two zones; that potentially chain different plugin:
  ```
  example.org {
      whoami
  }
  org {
      whoami
  }
  ```
  
  Note that most specific zone wins when a query comes in, so any example.org queries are going through the server defined for example.org above. The queries for .org are going to the other server.

  
  **Reverse Zones**
  
  Normally when you want to serve a reverse zone you’ll have to say something:
  ```
  0.0.10.in-addr.arpa {
      whoami
  }
  ```

  To make this easier CoreDNS just allows you to say:
  ``` 
  10.0.0.0/24 {
      whoami
  }
  ```


  This also works for CIDR (in the 1.0.0 release) zones:
  ```
  10.0.0.0/27 {
      whoami
  }
  ```


 **Non Default Protocols**
 
  Listening on TLS and for gRPC? Use:
  ```
  tls://example.org grpc://example.org {
      # ...
  }
  ```


  Specifying ports works in the same way, here when listening for gRPC packets.
  ```
  grpc://example.org:1443 {
      # ...
  }
  ```


## Configuration

There are various pieces that can be configured in CoreDNS. The first is determining which plugins you want to compile into CoreDNS. The binaries we provide have all plugins, as listed in plugin.cfg, compiled in. Adding or removing is easy, but requires a recompile of CoreDNS.

Thus most users use the Corefile to configure CoreDNS. When CoreDNS starts, and the -conf flag is not given, it will look for a file named Corefile in the current directory. That file consists of one or more Server Blocks. Each Server Block lists one or more Plugins. Those Plugins may be further configured with Directives.

The ordering of the Plugins in the Corefile does not determine the order of the plugin chain. The order in which the plugins are executed is determined by the ordering in plugin.cfg.

Comments in a Corefile are started with a #. The rest of the line is then considered a comment.


## Testing

Once you have a coredns binary, you can use the -plugins flag to list all the compiled plugins. Without a Corefile (See Configuration) CoreDNS will load the whoami plugin that will respond with the IP address and port of the client. So to test, we start CoreDNS to run on port 1053 and send it a query using dig:
```
$ ./coredns -dns.port=1053
.:1053
2018/02/20 10:40:44 [INFO] CoreDNS-1.0.5
2018/02/20 10:40:44 [INFO] linux/amd64, go1.10,
CoreDNS-1.0.5
linux/amd64, go1.10,
```
And from a different terminal window, a dig should return something similar to this:
```
$ dig @localhost -p 1053 a whoami.example.org

;; QUESTION SECTION:
;whoami.example.org.		IN	A

;; ADDITIONAL SECTION:
whoami.example.org.	0	IN	AAAA	::1
_udp.whoami.example.org. 0	IN	SRV	0 0 39368 .
```


## References: 
https://weng-albert.medium.com/coredns-basic-troubleshooting-resolving-common-issues-en-07c18cd7f8fe
https://coredns.io/2017/07/23/corefile-explained/

   
