# nerc-logging-improvements

## Config
```bash
oc apply -k cluster-scope/base
# wait for odf/loki operator to init
oc apply -k odf
oc apply -k loki
```

## Create secrets
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
```
