#Case 2 Egress with Service Entry & Strict outbound traffic policy

This case assumes that you have istio installed with default configurations and have not made any changes to the base installation. It also assumes that you have enabled Istio on the namespace your application is running on or the application pod has the Istio side injected. We will walkthrough a canonical repsentation of the traffic flow when a http call is made from an application. Istio is bound to hijack all the traffic coming out of the application and the destiny of the traffic is dependent upon policies of Envoy proxy.

##### Prerequisites & Configurations 
- Istio installed with default configurations.
- Istio is enabled on the namespace that these steps are going to be run on. 
- No Outbound policies (Service entry, Virtual Service, Gateways, Destination Rules)
- Check what's Istio's global outbound traffic policy by running the following command. If its `ALLOW_ANY` change it to `REGISTER_ONLY` by the following command
    `$ kubectl get configmap istio -n istio-system -o yaml | sed 's/mode: REGISTRY_ONLY/mode: ALLOW_ANY/g' | kubectl replace -n istio-system -f -`
- Please bear in mind that these changes take a minute or two to propagate 
- No internal or external 3rd party firewalls/proxies  manipulating traffic of the your k8s cluster 
- 

#####Figure depicting traffic flow
# ![alt text](https://gabbar-d2iq.s3-us-west-2.amazonaws.com/istio-diagrams/Istio-Egress-1.jpg)
#####Walkthrough
Unlike the previous case, In this case the global.outbound traffic is set to `REGISTER_ONLY`which means that Istio would only allow traffic to services that registered to the mesh. All the services living in the namespace would already be registered in Istio-service-registry. 

Service entry is typicaly used to register an external service to mesh. Thereby informing the mesh of external registered service that traffic can be routed to essentially making it a part of the mesh. Please note that if you have enabled mtls in your cluster. It's not going to garuntee automatic mtls between your application and external service.


1. An application makes a http request to `d2iq.com`
2. Envoy hijacks the traffic and checks to see if it know about domain that is being requested.
3. Since the `global.outbound.policy` is set to `REGISTRY_ONLY`, Envoy is going the check it's service registry to see if there any service entries.
4. Envoy finds `d2iq.com`  be an external service registered to the mesh and hence lets the traffic flow to `d2iq.com`
6. If you make a request to some other service which is not registered to the mesh. You would get a `404 page not found`