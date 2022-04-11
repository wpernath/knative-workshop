# knative-workshop
This is a simple demo project to demonstrate knative-serving. If you want to follow this demo, please use a recent OpenShift 4.8+ cluster (either CodeReady Containers or SNO). And install OpenShift Serverless.

## Using the Web UI
- Install person-service helm chart
- Click on "Make serverless"
- Add an environment variable APP_GREETING with some nice content
- Click on Edit Service to make the ksvc use the database:

```YAML
          envFrom:
            - prefix: DB_
              secretRef:
                name: ps1-db-pguser-ps1-db
```

- Demonstrate that it works now. 
- Wait a bit until it settled down

## A/B Deployments
- Go to the console
- Deploy a new version

```bash
kn service update ksvc-ps1 --image=quay.io/wpernath/person-service:v1.8.10-jvm --env APP_GREETING="We are now in JVM mode"
kn revision list
https --verify=no ....
```

- Tag the two (useable) revisions

```bash
kn service update ksvc-ps1 --tag=ksvc-ps1-00003=jvm
kn service update ksvc-ps1 --tag=ksvc-ps1-00002=native
kn revision list
```

- Now update the service to get only one or the other tag (A/B deployments)

```bash
kn service update ksvc-ps1 --traffic native=100,jvm=0
kn service update ksvc-ps1 --traffic native=0,jvm=100
```

## Canary Deployments
- The only difference to A/B is that we are going to have traffic splits between the old and the new version

```bash
kn service update ksvc-ps1 --traffic native=80,jvm=20
watch -n1 http ....
```

## Autoscaling
- New namespace
- Create the knative helm chart
- Install it twice with --set deployment.tag=jvm|native

```bash
helm package knative-chart
helm install ps1-native <chart-file>
helm install ps1-jvm <chart-file> --set deployment.tag=jvm

```
- Do load test with `hey`

```bash
hey -T application/json -c 16 -n 32000 <url>
```

