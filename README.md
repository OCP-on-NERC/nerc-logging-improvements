# nerc-logging-improvements

## Config
### Apply essential elements to cluster-scope
```bash
oc apply -k cluster-scope/base/
# wait for odf/loki operators to init before continuing
```

### Create and configure cluster storage
```bash
oc apply -k odf/
# wait for ocs-storagecluster to finish init. ~5-10 minutes
```

### Add loki to cluster storage
```bash
oc apply -k loki/base/backingstores/
oc apply -k loki/base/bucketclasses/
oc apply -k loki/base/storageclasses/
oc apply -k loki/base/objectbucketclaim/
oc apply -k loki/base/lokistack/
# wait for all to finish init. ~1-2 minutes
```

### Add Storage secrets
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

### Add logging
```bash
oc apply -k logging/

# create secret/lokistack-gateway-bearer-token created
oc -n openshift-logging create secret generic lokistack-gateway-bearer-token   --from-literal=token="$(oc -n openshift-logging get secret logcollector-token --template='{{.data.token | base64decode}}')"    --from-literal=ca-bundle.crt="$(oc -n openshift-logging get configmap openshift-service-ca.crt --template='{{index .data "service-ca.crt"}}')"
```

### Install min.io client
```bash
#https://min.io/docs/minio/linux/reference/minio-mc.html

S3_PUBLIC_HOST=$(oc -n openshift-storage get route/s3 -o jsonpath="{.spec.host}")

mc alias set logs https://$S3_PUBLIC_HOST $ACCESS_KEY_ID $AWS_SECRET_ACCESS_KEY
# view logs
mc ls logs/$BUCKET_NAME/infrastructure
```


