## Running on Openshift with Jenkins CI/CD

First, make sure you have an instance of Openshift setup and are logged in using `oc login`.

1. Create a new project:

```
$ oc new-project lab-c
```

2. Install Jenkins CI from OpenShift Catalog

Switch to `Developer` view > Click on `Add+` > Select `Developer Catalog` > Search for `Jenkins` (service with persistent storage) > Click `Instantiate Template` > Click `Create`.

3. Check access to Jenkins by opening its route:

```
$ oc get routes

NAME      HOST/PORT                                         PATH   SERVICES   PORT    TERMINATION     WILDCARD
jenkins   jenkins-lab-b.apps.myocp.foo.bar.baz          jenkins    <all>   edge/Redirect   None
```

4. Clone this repo and access the `openshift/` directoy

5. Create the application build&deploy template:


```
$ oc apply -f template-nodejs-postgresql.yaml
```

6. Create the BuildConfig JenkinsPipeline

```
$ oc apply -f nodejs-pipeline.yaml
```

7. Check the created BuildConfig

```
$ oc get bc

NAME                    TYPE              FROM   LATEST
nodejs-pipeline         JenkinsPipeline          0
``` 

7. Run the Pipeline

```
$ oc start-build nodejs-pipeline

build.build.openshift.io/nodejs-pipeline-1 started
```

8. Check the following resources

```
$ oc get deploymentconfig

NAME                    REVISION   DESIRED   CURRENT   TRIGGERED BY
jenkins                 1          1         1         config,image(jenkins:2)
nodejs-rest-http-crud   1          1         1         config,image(nodejs-rest-http-crud:latest)
postgresql              1          1         0         config,image(postgresql:12-el8)

$ oc get routes
NAME                    HOST/PORT                                                       PATH   SERVICES                PORT    TERMINATION     WILDCARD
jenkins                 jenkins-lab-b.apps.myocp.foo.bar.baz                                   jenkins                 <all>   edge/Redirect   None
nodejs-rest-http-crud   nodejs-rest-http-crud-lab-b.apps.myocp.foo.bar.baz                     nodejs-rest-http-crud   <all>                   None

$ oc get imagestream

NAME                            IMAGE REPOSITORY                                                                       TAGS     UPDATED
nodejs-rest-http-crud           image-registry.openshift-image-registry.svc:5000/lab-b/nodejs-rest-http-crud           latest   About a minute ago
nodejs-rest-http-crud-staging   image-registry.openshift-image-registry.svc:5000/lab-b/nodejs-rest-http-crud-staging   latest   55 seconds ago
```

*PS:* For development purposes, you can leverage this CI pipeline to prepare a staging image. This pipeline could be improved to support Continuous Deployment (deploy the generated artifacts on stage and prod environments). 
