# Egress with Egress Gateway 

#Case 2 Egress with Service Entry & Strict outbound traffic policy

This case assumes that you intend to route all of your external traffic through Engress gateway which is a stardard best practise. Also we will assume that istio installtion was done with default configurations and we have not made any changes to the base installation. We will also assume that you have enabled Istio on the namespace your application is running on or the application pod has the Istio side injected. We will walkthrough a canonical repsentation of the traffic flow when a http call is made from an application. Istio is bound to hijack all the traffic coming out of the application and the destiny of the traffic is dependent upon policies of Envoy proxy. 


Consider an organization that has a strict security requirement that all traffic leaving the service mesh must flow through a set of dedicated nodes. These nodes will run on dedicated machines, separated from the rest of the nodes running applications in the cluster. These special nodes will serve for policy enforcement on the egress traffic and will be monitored more thoroughly than other nodes.

Another use case is a cluster where the application nodes don’t have public IPs, so the in-mesh services that run on them cannot access the Internet. Defining an egress gateway, directing all the egress traffic through it, and allocating public IPs to the egress gateway nodes allows the application nodes to access external services in a controlled way. 

##### Prerequisites & Configurations 
- Istio installed with default configurations.
- Istio is enabled on the namespace that these steps are going to be run on. 
- No Outbound policies (Service entry, Virtual Service, Gateways, Destination Rules)
- Check what's Istio's global outbound traffic policy by running the following command. If its `ALLOW_ANY` change it to `REGISTER_ONLY` by the following command
    `$ kubectl get configmap istio -n istio-system -o yaml | sed 's/mode: REGISTRY_ONLY/mode: ALLOW_ANY/g' | kubectl replace -n istio-system -f -`
- Please bear in mind that these changes take a minute or two to propagate 
- No internal or external 3rd party firewalls/proxies  manipulating traffic of the your k8s cluster 
- Defined routing rules(virtual services and destination rules) to route the traffic to Egreess-Gateway and then route to the desired service from Egress gateway.

#####Figure depicting traffic flow
# ![alt text](https://gabbar-d2iq.s3-us-west-2.amazonaws.com/istio-diagrams/Istio-Egress-1.jpg)
#####Walkthrough
Since the global.outbound traffic is set to `REGISTER_ONLY`which means that Istio would only allow traffic to services that registered to the mesh. All the services living in the namespace would already be registered in Istio-service-registry. Once Istio resolves the target service and identifies it to be a part of the mesh. When Envoy receives a request for routing traffic, It first tries to resolve the service to see the requested service is a registered in the mesh. Once it finds the service then it tries to look up if there are any `intelligent routing rules`. When it find the rules then Envoy abides by the rules(Virtual service, Desitnation rules) defined. If not, Envoy would try to initiate contact direcly to the service.  

1. An application makes a http request to `d2iq.com`
2. Envoy hijacks the traffic and checks to see if it know about domain that is being requested.
3. Since the `global.outbound.policy` is set to `REGISTRY_ONLY`, Envoy is going the check it's service registry to see if there any service entries.
4. Envoy finds `d2iq.com`  be an external service registered to the mesh and hence lets the traffic flow to `d2iq.com`
7. Envoy looks up if there are any routing rules, once it finds the rules, it routes the traffic to Egress-gateway and Egress gaetway in turn routes the traffic to the external service. 
