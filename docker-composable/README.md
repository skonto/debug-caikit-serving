
Adapted from https://github.com/opendatahub-io/caikit-tgis-serving/tree/main/test.
Queue proxy adapted to start a reverse proxy that calls caikit not 127.0.0.1 host (in docker compose you can access other services via their name).

run to setup the containers

```bash
docker compose up --no-build

directly to the caikit model

grpcurl -d '{"text": "At what temperature does liquid Nitrogen boil?"}' -H 'mm-model-id: flan-t5-small-caikit' --plaintext  127.0.0.1:8085 caikit.runtime.Nlp.NlpService/TextGenerationTaskPredict
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


via queue proxy

grpcurl -d '{"text": "At what temperature does liquid Nitrogen boil?"}' -H 'mm-model-id: flan-t5-small-caikit' --plaintext  127.0.0.1:8013 caikit.runtime.Nlp.NlpService/TextGenerationTaskPredict
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

caikit 8080 http port

curl  --json '{  "model_id": "flan-t5-small-caikit",
"inputs": "At what temperature does liquid Nitrogen boil?"}' 127.0.0.1:8080/api/v1/task/text-generation
{"generated_text": "74 degrees F", "generated_tokens": 5, "finish_reason": "EOS_TOKEN", "producer_id": {"name": "Text Generation", "version": "0.1.0"}, "input_token_count": 10, "seed": null}


Note: you can get a copy of the flan-t5-small-caikit model with:

docker create quay.io/opendatahub/modelmesh-minio-examples:caikit-flan-t5

cp the container id

docker cp <id>:/data1/modelmesh-example-models/llm/models/flan-t5-small-caikit - > flan.tar
untar under ./models/

```
