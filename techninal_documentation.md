# Technical Documentation for NERC-logging-improvements


## Setting up OpenShift Clusters
---
CLI:
1. log in: `oc login --server=<OPENSHIFT_API_URL> --token=<YOUR_TOKEN>`
2. set active project: `oc project <PROJECT_NAME>`
>or create project: `oc new-project <PROJECT_NAME>`
3. `oc apply -k cluster-scope/base/`
*wait for odf/loki operators to init before continuing*


## Configure Storage in Clusters
---
CLI:
1. `oc apply -k odf/`
GUI:
*may need to create storagecluster on GUI interface. need to find how to set this up in CLI*
Admin > Installed Operators > 'Create StorageSystem'
1. Backing storage
Deployment type = Full deployment > Backing strorage type = Use an existing StorageClass > StorageClass = gp2 > Click Next
2. Capacity and nodes
Select at least 3 nodes > Click Next
3. Security and network
Click Next 
4. Review and create
Click Create StorageSystem
*If you get an Error 404:Page Not Found, ignore it*
Wait for ocs-storagecluster to finish init. ~5-10 minutes


## Add loki to cluster storage
---
`oc apply -k loki/base/backingstores/`
`oc apply -k loki/base/bucketclasses/`
`oc apply -k loki/base/storageclasses/`
`oc apply -k loki/base/objectbucketclaims/`
`oc apply -k loki/base/lokistacks/`
# wait for all to finish init. ~1-2 minutes


## Add Storage secrets
---
```bash
ACCESS_KEY_ID=$(oc -n openshift-logging get secrets/logging-objectbucketclaim -o jsonpath={.data.AWS_ACCESS_KEY_ID} | base64 -d)
AWS_SECRET_ACCESS_KEY=$(oc -n openshift-logging get secrets/logging-objectbucketclaim -o jsonpath={.data.AWS_SECRET_ACCESS_KEY} | base64 -d)
BUCKET_NAME=$(oc -n openshift-logging get configmap/logging-objectbucketclaim -o jsonpath={.data.BUCKET_NAME})
BUCKET_HOST=$(oc -n openshift-logging get configmap/logging-objectbucketclaim -o jsonpath={.data.BUCKET_HOST})

oc -n openshift-logging create secret generic thanos-object-storage \
    --from-literal="access_key_id=${ACCESS_KEY_ID}" \
    --from-literal="access_key_secret=${AWS_SECRET_ACCESS_KEY}" \
    --from-literal="bucketnames=${BUCKET_NAME}" \
    --from-literal="endpoint=https://${BUCKET_HOST}"

# wait for all to finish init. ~10 minutes
```

## Add logging
---
```bash
oc apply -k logging/

oc -n openshift-logging create secret generic lokistack-gateway-bearer-token   --from-literal=token="$(oc -n openshift-logging get secret logcollector-token --template='{{.data.token | base64decode}}')"    --from-literal=ca-bundle.crt="$(oc -n openshift-logging get configmap openshift-service-ca.crt --template='{{index .data "service-ca.crt"}}')"
```


## Enable logging on openshift console
---
GUI:
Openshift console > Installed Operators > Red Hat OpenShift Logging > console plugin > enable console plugin.
*Will need to wait for console to signal a reload. Takes a while*


## Create a Ceph user with read-only RBAC on source Cluster
---
*ive not used these commands. commands may not be entirly correct. found on stackoverflow*
CLI:
`oc exec -it <CEPH_POD_NAME> -- radosgw-admin user create --uid="sourceuser" --display-name="Source User"`


## Create a Ceph user with write-only RBAC on destination Cluster
---
*ive not used these commands. commands may not be entirly correct. found on stackoverflow*
CLI:
`oc exec -it <CEPH_POD_NAME> -- radosgw-admin user create --uid="destinationuser" --display-name="Destination User"`


## Configure `s3cmd`
---
CLI:
on source cluster
s3cmd --configure
- S3 Endpoint: (Endpoint of Cluster A's Ceph RGW)
- Access Key: (Access key of "sourceuser" created above)
- Secret Key: (Secret key of "sourceuser" created above)


### Make note of the .s3cfg file which is generated. This will be used to configure s3cmd on other containers or pods.

`exit` # To exit from the pod


## Store s3cmd configuration for future use
`oc create configmap s3cmd-config --from-file=$HOME/.s3cfg`


## Create s3cmd runner pod to perform the syncs using the saved configuration
`oc run s3cmd-runner --image=alpine --restart=OnFailure -- sh -c "apk add --no-cache s3cmd && s3cmd sync s3://source-bucket/ s3://destination-bucket/ --config /config/.s3cfg"`
`oc set volume pod/s3cmd-runner --add -m /config --configmap-name=s3cmd-config`


# Create a shell script for the s3cmd sync command
`echo '#!/bin/sh s3cmd sync s3://source-bucket/ s3://destination-bucket/ --config /config/.s3cfg' > s3cmd_sync.sh`

## Deploy this script within a config map in OpenShift
`oc create configmap s3cmd-script --from-file=s3cmd_sync.sh`

## Define a CronJob to run the script at regular intervals (for example, every day at midnight)
```bash
cat <<EOF | oc apply -f -
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: s3cmd-sync-cron
spec:
  schedule: "0 0 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: s3cmd
            image: alpine
            command: ["/bin/sh", "/script/s3cmd_sync.sh"]
            volumeMounts:
            - name: config
              mountPath: /config
            - name: script
              mountPath: /script
          volumes:
          - name: config
            configMap:
              name: s3cmd-config
          - name: script
            configMap:
              name: s3cmd-script
          restartPolicy: OnFailure
EOF
```

# Monitoring and Validations
## Use oc logs to validate CronJob logs
`oc logs job/<NAME_OF_THE_JOB_FROM_CRONJOB>`

## Periodically compare the source and destination buckets' contents to ensure integrity
`s3cmd ls s3://source-bucket/`
`s3cmd ls s3://destination-bucket/`


Define policies or automation scripts to delete older logs from the source bucket after they've been confirmed to exist in the destination bucket.
This can be done using s3cmd as well with specific date filters and other options.


