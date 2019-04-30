# OpenShif Tekton Demo
Access OpenShift integrated registry using [tekton pipeline](https://github.com/tektoncd/pipeline) on OpenShift 4 cluster.


## Prerequisites:

* ##### V2 Schema
  Make sure integrated registry has enabled V2 API schema.
  On OpenShift4 it could enabled by
  ```
  oc set env \
  deployment/image-registry \
  REGISTRY_MIDDLEWARE_REPOSITORY_OPENSHIFT_ACCEPTSCHEMA2=true \
  -n openshift-image-registry 
  ```   
  
* ##### Registry access permissions
  A valid registry credentials(username, password or token) are required in order to access the registry.
  Every `project` in the OpenShift has a `ServiceAccount` named `builder` which contains a `secret` of type `kubernetes.io/dockercfg`. 
  This `secret` can be used to `pull` and `push` an image from/to an integrated registry.

  However to [reccomonded](https://docs.openshift.com/container-platform/4.0/registry/accessing-the-registry.html) way is to add `registry-editor` role to `user` or `ServiceAccount (SA)` 


  1. Assigning role to user
  ```shell
  oc policy add-role-to-user registry-editor <user>
  ```
  2. Assigning role to SA
  ```shell
  oc policy add-role-to-user registry-editor -z <SA>
  ```
  Notes: 
  - Requires `admin` privileges to execute above commands. 
  - The `tls` verification should be ignored while interacting with `image-registry` service which has self signed certificates.
  - In OpenShift 3.x, every `namespace` has `builder` SA and registry service name is `docker-registry` which runs under `default` namespace/project.  


## Examples:

##### 1. Build sample application and push image to OpenShift registry
This [example](os-build-push-task.yaml) use `kaniko` to build an container image then push it the OpenShift registry using `builder` `ServiceAccount`
```
oc create project demo
oc apply -f os-build-push-task.yaml
```

##### 2. Copy a image and push it to OpenShift registry 
This [example](os-copy-puos-copy-push-task.yamlsh-task.yaml) uses `scopeo` to copy an image from `openshift` namespace then push it to the OpenShift registry using `tkn-pipeline` `ServiceAccount`
```
oc create project demo
oc create sa tkn-pipeline
oc policy add-role-to-user registry-editor -z tkn-pipeline
oc apply -f os-copy-push-task.yaml
```

To verify the image, execute 
```
oc get is
``` 
  
  

