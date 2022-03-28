# ConfigMap-functionality
Understanding how configmap mounting affects mutability & immutability of data

## Installation:

In a cluster deploy the four files in the following order:

1. secrets.yaml
2. mysql-deployment.yaml
3. config-map.yaml
4. wordpress-deployment.yaml

## Use Case:

### Mutability:

In this case we will be utilizing the wordpress-deployment.yaml file:

        - name: my-blog-nginx-site
          mountPath: /etc/nginx/conf.d/
          
          
At the line number 82 & 83 of the wordpress-deployment.yaml file, we can see the config map mounted on a directory named "/etc/nginx/conf.d"

Now, In a scenrio, where we want to edit the configurational parameters of our config-map like from "fastcgi_buffer_size 32k;" to "fastcgi_buffer_size 48k;". We won't be needing to manually make roll-out of our deployement to make the changes take affects nor our deployment will be performing the rolling-update. As the changes made in the config-map are automaitcally synced by the kubelet after 2 to 5 minutes of interval.

The kubelet has a parameter called "configMapAndSecretChangeDetectionStrategy" in its configuration which uses the TTL method for periodic sycning & it syncs cm's through kube-api server & then make changes in the POD accordingly.


### Immutability:

1. To make our config-map immutable we will use the following parameter in our config-map:

          apiVersion: v1
          kind: ConfigMap
          metadata:
            ...
          data:
            ...
          immutable: true

2. ConfigMaps consumed as environment variables are not updated automatically and require a pod restart or deployment roll-out.

### Example:

          apiVersion: v1
          kind: Pod
          metadata:
            name: dapi-test-pod
          spec:
            containers:
              - name: test-container
                image: k8s.gcr.io/busybox
                command: [ "/bin/sh", "-c", "env" ]
                env:
                  # Define the environment variable
                  - name: SPECIAL_LEVEL_KEY
                    valueFrom:
                      configMapKeyRef:
                        # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
                        name: special-config
                        # Specify the key associated with the value
                        key: special.how
            restartPolicy: Never


Making the configmap immutable has the following advantages:

* protects you from accidental (or unwanted) updates that could cause applications outages
* improves performance of your cluster by significantly reducing load on kube-apiserver, by closing watches for ConfigMaps marked as immutable.


## Reading Material:

https://kubernetes.io/docs/concepts/configuration/configmap/#mounted-configmaps-are-updated-automatically --- Config-map mutability & immutability

https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#define-a-container-environment-variable-with-data-from-a-single-configmap -- configuring configmap as envoirnment variable

https://github.com/kubernetes/kubernetes/issues/30189 -- Kubelet sync interval
