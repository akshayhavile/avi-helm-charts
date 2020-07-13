### HTTPRule

The HTTPRule CRD is primarily targetted for the developers. While the path matching rules in the Ingress/Route objects would define
traffic routing rules to the microservices, the HTTPRule CRD can be used as a complimentary object to control additional layer 7
properties like: algorithm, hash, tls re-encrypt use cases.

A sample HTTPRule object looks like this:

    apiVersion: ako.k8s.io/v1alpha1
    kind: HTTPRule
    metadata:
       name: my-http-rules
       namespace: purple-l7
    spec:
      hostrule: default/secure-waf-policy
      paths:
      - target: /foo
        loadBalancerPolicy:
          algorithm: LB_ALGORITHM_CONSISTENT_HASH
          hash: LB_ALGORITHM_CONSISTENT_HASH_SOURCE_IP_ADDRESS
        tls: ## This is a re-encrypt to pool
          type: reencrypt # Mandatory [re-encrypt]
          sslProfile: "System-Default"

__NOTE__ : The HTTPRule only applies to paths in the Ingress/Route objects which are specified in the same namespace as the CRD.

### Specific usage of the HTTPRule CRD

The HTTPRule CRD does not have any Avi specific semantics. Hence the developers are free to express their preferences using this CRD
without any knowledge of the Avi objects. Each HTTPRule CRD must be bound to a HostRule CRD to subscribe to rules for a specific FQDN.

A HTTPRule CRD would be `Rejected` if the corresponding HostRule CRD does not exist. 

#### Express loadbalancer alogrithm

The loadbalancer policies are a predefined set of values which the user can choose from. Presently the following values are supported for
loadbalancer policy:

      - LB_ALGORITHM_CONSISTENT_HASH
      - LB_ALGORITHM_CORE_AFFINITY
      - LB_ALGORITHM_FASTEST_RESPONSE
      - LB_ALGORITHM_FEWEST_SERVERS
      - LB_ALGORITHM_FEWEST_TASKS
      - LB_ALGORITHM_LEAST_CONNECTIONS
      - LB_ALGORITHM_LEAST_LOAD
      - LB_ALGORITHM_NEAREST_SERVER
      - LB_ALGORITHM_RANDOM
      - LB_ALGORITHM_ROUND_ROBIN
      - LB_ALGORITHM_TOPOLOGY

The way one could configure the loadbalancer policy for a given ingress path is as follows:

      - target: /foo 
        loadBalancerPolicy:
          algorithm: LB_ALGORITHM_CONSISTENT_HASH
          
This rule is applied all paths matching `/foo` and subsets of `/foo/xxx`

More about pool algorithm can be found [here](https://avinetworks.com/docs/18.1/load-balancing-algorithms/).


#### Reencrypt traffic to the services

While AKO can terminate TLS traffic, it also provides and option where the users can choose to re-encrypt the traffic between the Avi SE and the
backend application server. The following option is provided for `reencrypt`:

        tls: ## This is a re-encrypt to pool
          type: reencrypt # Mandatory [re-encrypt]
          
If the `sslProfile` is not selected, the AKO selects the default configured SSL profile in Avi to exchange the tls parameters like TLS versions,
ciphers etc.

As a further enhancement, the HTTPRule CRD would also allow specification of a destinationCa, that could be used as a pki profile to validate
the server certificates.

#### Status Messages

The status messages are used to give instanteneous feedback to the users about the whether a HTTPRule CRD was `Accepted` or `Rejected`.

Since the HTTPRule CRD always refers to a HostRule CRD, it's mandatory to ensure the HostRule CRD is also in `Accepted` condition for the HTTPRule
CRD to apply.

##### Accepted HTTPRule

    $ kubectl get httprule
    NAME            HOSTRULE                     STATUS     AGE
    my-http-rules   default/secure-waf-policy    Accepted   5h34m

