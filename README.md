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
oc set resources dc nexus-service --requests=memory=1Gi,cpu=1
```
6. Create a PVC for nexus-service
```bash
oc set volume dc/nexus-service --add --overwrite --name=nexus-service-volume --mount-path=/nexus-data/ --type persistentVolumeClaim --claim-name=nexus-service-pvc --claim-size=5Gi
```
7. Rollout deployment
```bash
oc rollout resume dc nexus-service
```
8. Fetch admin password from `/nexus-data/admin.password` and use it to access and configure your Nexus service.

## SonarQube
1. Create a new project for sonarqube
```bash
oc new-project sonarqube-service --display-name "sonarqube service"
```
2. Deploy a persitent postgresql database using the template `postgresql-persistent`
```bash
oc new-app --template=postgresql-persistent --param POSTGRESQL_USER=sonar --param POSTGRESQL_PASSOWRD=sonar --param POSTGRESQL_DATABASE=sonar --param VOLUME_CAPACITY=4Gi --labels=app=sonar-database --as-deployment-config=true
```
3. Deploy SonarQube using `sonarqube:lts` from dockerhub
```bash
oc new-app --docker-image=sonarqube:lts --env=SONARQUBE_JDBC_USERNAME=sonar --env=SONARQUBE_JDBC_PASSWORD=sonar --env=SONARQUBE_JDBC_URL=jdbc:postgresql://postgresql/sonar --as-deployment-config=true
oc rollout pause dc sonarqube
```
4. Expose SonarQube by creating a route for it
```bash
oc expose service sonarqube-service
```
5. Set enough resources to your sonarqube-service (memory, volume and cpu)
```bash
oc set resources dc sonarqube-service --requests=memory=1Gi,cpu=1
```
6. Create a PVC
```bash
oc set volume dc/sonarqube-service --add --overwrite --name=sonarqube-service-volume --mount-path=/opt/sonarqube/data/ --type persistentVolumeClaim --claim-name=sonarqube-service-pvc --claim-size=3Gi
```
7. Rollout deployment
```bash
oc rollout resume dc sonarqube-service
```


## Jenkins 
1. Create new jenkins-service project
```bash
oc new-project jenkins-service --display-name "Jenkins Service"
```
2. Create new application from the template `jenkins-persistent`with pvc
```bash
oc new-app --template=jenkins-persistent --VOLUME_CAPACITY=2Gi --as-deployment-config=true
oc rollout pause dc jenkins-service
```
3. Set enough resources to your sonarqube-service (memory, volume and cpu)
```bash
oc set resources dc jenkins-service --requests=memory=1Gi,cpu=1
```
5. Rollout deployment
```bash
oc rollout resume dc jenkins-service
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
2. https://docs.openshift.com/container-platform/4.1/openshift_images/using_images/images-other-jenkins-agent.html#images-other-jenkins-agent-pod-retention_images-other-jenkins-agent
3. https://github.com/openshift/origin/blob/master/examples/jenkins/jenkins-persistent-template.json
4. https://github.com/openshift
5. https://github.com/openshift/openshift-docs
6. https://www.openshift.com/blog/deploy-jenkins-pipelines-in-openshift-4-with-openshift-container-storage-4
7. https://github.com/openshift/origin/blob/master/examples/jenkins/README.md
