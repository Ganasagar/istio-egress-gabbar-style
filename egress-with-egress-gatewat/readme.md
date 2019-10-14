#Case 1 Egress with no routing policies 

This case assumes that you have istio installed with the defaults configurations and have not made any changes to the base installation. If you have enabled Istio on specific namespace the application pod has the Istio side car running, then Istio is bound to hijack all the traffic coming out of the application and the destiny of the traffic is dependent upon policies of Envoy proxy. By default Istio configures Envoy proxy to passthrough request for any unknown services as in services that are not defined in the cluster through an installation option `global.outboundTrafficPolicy.mode` this is set to `ALLOW_ANY` in the default installation. 

##### Prerequisites
- Istio installed with default configurations 
- Istio is enabled on the namespace these steps are going to run on.
- No Outbound policies (Service entry, Virtual Service, Gateways, Destination Rules) 
- Deploy a sleep application it should run the side car proxy along with it.
- Validate Istio configured the global outbound traffic policy to allow any by running the following command. It should result the entries listed in the configmap like shown below 
    `$ kubectl get configmap istio -n istio-system -o yaml | grep -o "mode: ALLOW_ANY"`
    `mode: ALLOW_ANY`
- No internal or external firewalls/proxies  manipulating traffic of the your k8s cluster 


#####Figure depicting traffic flow
# ![alt text](https://gabbar-d2iq.s3-us-west-2.amazonaws.com/istio-diagrams/Istio-Egress-1.jpg)
#####Walkthrough
As we already understand how traffic flow is different in Istio when compared to Kubernetes. We know that the traffic is expected to be hijacked by envoy and Envoys routes the traffic to the services defined in Istio's service registry(This of this as Istio own directory Telephone directory)

1. An application makes a http based request 
2. Envoy hijacks the traffic and checks to see if it know about domain that is being requested.
3. Upon not being able to find the service envoy would deem the service as unknown service beyond the mesh 
4. Since the `global.outbound.policy` is set to allow any, Envoy is going the let request pass through the mesh.
5. If the `global.outbound.policy` is set to `REGISTRY_ONLY`, this would deny envoy proxies from communicating with any services beyond the mesh.
6. In such a case egress traffic would fail to exit the mesh, you would typically get a `404 Error` while you make a call to an external service
