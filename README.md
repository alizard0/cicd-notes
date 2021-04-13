# cicd-notes
Some notes for setting up a ci/cd with nexus, sonarq and jenkins.

> Jenkins enables developers around the world to reliably build, test, and deploy their software.

> SonarQube catches bugs and vulnerabilities in your app, with thousands of automated Static Code Analysis rules.

> Nexus manages binaries and builds artifacts across your software supply chain.


## Nexus
1. Official docker image `sonatype/nexus3:latest`
2. Create a new project for nexus-service
```bash
oc new-project nexus-service --display-name "nexus service"
```
3. Deploy nexus container image
```bash
oc new-app sonatype/nexus3:latest --name=nexus-service --as-deployment-config=true
```
4. Expose nexus service
```bash
oc expose svc nexus-service
oc rollout pause dc nexus-service
```
5. Set enough resources to your nexus-service (memory, volume and cpu)
```bash
oc set resources dc nexus-service --limits=memory=2Gi,cpu=3
```
6. Create a PVC for nexus-service
```bash
oc set volume dc/nexus-service --add --overwrite --name=nexus-service-volume --mount-path=/nexus-data/ --type persistentVolumeClaim --claim-name=nexus-service-pvc --claim-size=5Gi
```
7. Rollout deployment
```bash
oc rollout resume dc nexus-service
```



### Notes
1. Persistent Volume Claim is a request object for creating Storage on OpenShift.
```yml
apiVersion: "v1"
kind: "Pod"
metadata:
  name: "mypod"
  labels:
    name: "frontendhttp"
spec:
  containers:
    -
      name: "myfrontend"
      image: "nginx"
      ports:
        -
          containerPort: 80
          name: "http-server"
      volumeMounts:
        -
          mountPath: "/var/www/html"
          name: "pvol"
  volumes:
    -
      name: "pvol"
      persistentVolumeClaim:
        claimName: "claim1"
```


### Links
1. https://docs.openshift.com/enterprise/3.1/dev_guide/persistent_volumes.html
