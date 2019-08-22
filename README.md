openjdk-cron add image stream to openshift project

`oc import-image openjdk18-openshift --from=registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift --confirm -n openshift`

create project

`oc new-project cj --description="My Test Cronjob application!"`

create build

`oc new-build --image-stream=openshift/openjdk18-openshift:latest --binary=true --name=cronapp`

create a new image using build config of jar + base openjdk image and store in project image stream

`oc start-build cronapp --from-dir=./deployment/deployments/`

import template to project

`oc create -f ./openjdk-cron.yaml`

create a service account

`oc create serviceaccount robot`

create the cronjob-admin cluster role (view + manage cronjobs)

`oc create -f cronjob_admin.yaml`

give robot service account cronjob-admin (view + manage cronjobs)

`oc adm policy add-role-to-user cronjob-admin -z robot`

get token from secret

```
oc get secrets
oc describe secret <robot token name>
```

```
oc logout

# login as robot
oc login --token=<robot token>
```

create cronjob from template (starts the cronjob)

`oc process openjdk-cron -p CRONJOB_NAME=cronapp -p IMAGE_STREAM_NAME="cj/cronapp:latest" -p CRONJOB_SCHEDULE="*/2 * * * *" | oc create -f -`

```
[praspa@tools openjdk-cron]$ oc whoami
system:serviceaccount:cj:robot

[praspa@tools openjdk-cron]$ oc get cj
NAME      SCHEDULE      SUSPEND   ACTIVE    LAST SCHEDULE   AGE
cronapp   */2 * * * *   False     0         1m              8m


[praspa@tools openjdk-cron]$ oc get pods
NAME                       READY     STATUS      RESTARTS   AGE
cronapp-1566443280-zqcwn   0/1       Completed   0          4m
cronapp-1566443400-f9tqw   0/1       Completed   0          2m
cronapp-1566443520-ncr8z   0/1       Completed   0          49s
```


