Adapted from https://github.com/opendatahub-io/caikit-tgis-serving/blob/main/demo/kserve/deploy-remove.md.

Install Serverless and ServiceMesh

Just follow instructions from https://github.com/reToCode/knative-kserve

Install opendatahub operator and apply the following

```bash
apiVersion: kfdef.apps.kubeflow.org/v1
kind: KfDef
metadata:
  name: opendatahub
  namespace: openshift-operators
spec:
  applications:
    - kustomizeConfig:
        repoRef:
          name: manifests
          path: odh-common
      name: odh-common
    - kustomizeConfig:
        repoRef:
          name: manifests
          path: kserve
      name: kserve
  repos:
    - name: manifests
      uri: https://api.github.com/repos/opendatahub-io/odh-manifests/tarball/master
  version: master

ACCESS_KEY_ID=admin
SECRET_ACCESS_KEY=password
MINIO_NS=minio

oc new-project ${MINIO_NS}
oc apply -f ./minio.yaml -n ${MINIO_NS}
sed "s/<minio_ns>/$MINIO_NS/g" ./custom-manifests/minio/minio-secret.yaml | tee ./minio-secret-current.yaml | oc -n ${MINIO_NS} apply -f -
sed "s/<minio_ns>/$MINIO_NS/g" ./custom-manifests/minio/serviceaccount-minio.yaml | tee ./serviceaccount-minio-current.yaml | oc -n ${MINIO_NS} apply -f -

export TEST_NS=kserve-demo
oc new-project ${TEST_NS}

oc apply -f serviceaccount-minio-current.yaml -n ${TEST_NS}

oc apply -f minio-secret-current.yaml -n ${TEST_NS}

oc apply -f runtime.yaml -n ${TEST_NS}

oc apply -f inference-svc.yaml -n ${TEST_NS}

$  grpcurl -d '{"text": "At what temperature does liquid Nitrogen boil?"}' -H 'mm-model-id: flan-t5-small-caikit' --insecure flan-t5-small-caikit-predictor-kserve-demo.apps.ci-ln-h2v45ck-76ef8.origin-ci-int-aws.dev.rhcloud.com:443  caikit.runtime.Nlp.NlpService/TextGenerationTaskPredict
{
"generated_text": "74 degrees F",
"generated_tokens": "5",
"finish_reason": "EOS_TOKEN",
"producer_id": {
  "name": "Text Generation",
  "version": "0.1.0"
},
"input_token_count": "10"
}

$ curl -k -d '{"text": "At what temperature does liquid Nitrogen boil?"}'  https://flan-t5-small-caikit-predictor-kserve-demo.apps.ci-ln-h2v45ck-76ef8.origin-ci-int-aws.dev.rhcloud.com:443/api/v1/task/text-generation
stream error: stream ID 21; INTERNAL_ERROR; received from peer

$  grpcurl -d '{"text": "At what temperature does liquid Nitrogen boil?"}' -H 'mm-model-id: flan-t5-small-caikit' --insecure flan-t5-small-caikit-predictor-kserve-demo.apps.ci-ln-h2v45ck-76ef8.origin-ci-int-aws.dev.rhcloud.com:443  caikit.runtime.Nlp.NlpService/TextGenerationTaskPredict
Error invoking method "caikit.runtime.Nlp.NlpService/TextGenerationTaskPredict": rpc error: code = Unavailable desc = failed to query for service descriptor "caikit.runtime.Nlp.NlpService": unexpected HTTP status code received from server: 502 (Bad Gateway); transport: received unexpected content-type "text/plain; charset=utf-8"
```
