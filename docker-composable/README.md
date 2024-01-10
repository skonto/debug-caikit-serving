
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

Failing the connection:

```
$ curl -v --http2  -d '{"text": "At what temperature does liquid Nitrogen boil?"}' -H 'Content-type: text/plain'  localhost:8013/api/v1/task/text-generation 
*   Trying 127.0.0.1:8013...
* Connected to localhost (127.0.0.1) port 8013 (#0)
> POST /api/v1/task/text-generation HTTP/1.1
> Host: localhost:8013
> User-Agent: curl/7.82.0
> Accept: */*
> Connection: Upgrade, HTTP2-Settings
> Upgrade: h2c
> HTTP2-Settings: AAMAAABkAAQCAAAAAAIAAAAA
> Content-type: text/plain
> Content-Length: 58
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 101 Switching Protocols
< Connection: Upgrade
< Upgrade: h2c
* Received 101
* Using HTTP2, server supports multiplexing
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Connection state changed (MAX_CONCURRENT_STREAMS == 250)!
< HTTP/2 502 
< content-type: text/plain; charset=utf-8
< x-content-type-options: nosniff
< content-length: 269
< date: Wed, 10 Jan 2024 09:01:51 GMT
< 
net/http: HTTP/1.x transport connection broken: malformed HTTP response "\x00\x00\x1e\x04\x00\x00\x00\x00\x00\x00\x03\x7f\xff\xff\xff\x00\x04\x00@\x00\x00\x00\x05\x00@\x00\x00\x00\x06\x00\x00@\x00\xfe\x03\x00\x00\x00\x01\x00\x00\x04\b\x00\x00\x00\x00\x00\x00?\x00\x01"
* Connection #0 to host localhost left intact

$ grpcurl -d '{"text": "At what temperature does liquid Nitrogen boil?"}' -H 'mm-model-id: flan-t5-small-caikit' --plaintext  127.0.0.1:8013 caikit.runtime.Nlp.NlpService/TextGenerationTaskPredict 
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

#Logs from the QP:

...
2024/01/09 22:14:22 http2: server read frame SETTINGS flags=ACK len=0
{"severity":"ERROR","timestamp":"2024-01-09T22:14:22.163972306Z","logger":"fallback-logger.queueproxy","caller":"network/error_handler.go:33","message":"error reverse proxying request; sockstat: sockets: used 9\nTCP: inuse 2 orphan 0 tw 0 alloc 64 mem 172\nUDP: inuse 1 mem 556\nUDPLITE: inuse 0\nRAW: inuse 0\nFRAG: inuse 0 memory 0\n","commit":"a826aa4-dirty","knative.dev/key":"kserve-demo/flan-t5-small-caikit-predictor-00001","knative.dev/pod":"flan-t5-small-caikit-predictor-00001-deployment-d7d684947-ll5nt","error":"net/http: HTTP/1.x transport connection broken: malformed HTTP response \"\\x00\\x00\\x1e\\x04\\x00\\x00\\x00\\x00\\x00\\x00\\x03\\x7f\\xff\\xff\\xff\\x00\\x04\\x00@\\x00\\x00\\x00\\x05\\x00@\\x00\\x00\\x00\\x06\\x00\\x00@\\x00\\xfe\\x03\\x00\\x00\\x00\\x01\\x00\\x00\\x04\\b\\x00\\x00\\x00\\x00\\x00\\x00?\\x00\\x01\"","stacktrace":"knative.dev/pkg/network.ErrorHandler.func1\n\tknative.dev/pkg@v0.0.0-20231211072236-4914c472e81a/network/error_handler.go:33\nnet/http/httputil.(*ReverseProxy).ServeHTTP\n\tnet/http/httputil/reverseproxy.go:475\nknative.dev/serving/pkg/queue.(*appRequestMetricsHandler).ServeHTTP\n\tknative.dev/serving/pkg/queue/request_metric.go:199\nknative.dev/serving/pkg/queue/sharedmain.mainHandler.ProxyHandler.func3\n\tknative.dev/serving/pkg/queue/handler.go:76\nnet/http.HandlerFunc.ServeHTTP\n\tnet/http/server.go:2136\nknative.dev/serving/pkg/queue/sharedmain.mainHandler.ForwardedShimHandler.func4\n\tknative.dev/serving/pkg/queue/forwarded_shim.go:50\nnet/http.HandlerFunc.ServeHTTP\n\tnet/http/server.go:2136\nknative.dev/serving/pkg/http/handler.(*timeoutHandler).ServeHTTP.func4\n\tknative.dev/serving/pkg/http/handler/timeout.go:118"}
2024/01/09 22:14:22 http2: server encoding header ":status" = "502"
2024/01/09 22:14:22 http2: server encoding header "content-type" = "text/plain; charset=utf-8"
2024/01/09 22:14:22 http2: server encoding header "x-content-type-options" = "nosniff"
2024/01/09 22:14:22 http2: server encoding header "content-length" = "269"
2024/01/09 22:14:22 http2: server encoding header "date" = "Tue, 09 Jan 2024 22:14:22 GMT"
2024/01/09 22:14:22 http2: Framer 0xc0000e7260: wrote HEADERS flags=END_HEADERS stream=1 len=77
2024/01/09 22:14:22 http2: Framer 0xc0000e7260: wrote DATA flags=END_STREAM stream=1 len=269 data="net/http: HTTP/1.x transport connection broken: malformed HTTP response \"\\x00\\x00\\x1e\\x04\\x00\\x00\\x00\\x00\\x00\\x00\\x03\\x7f\\xff\\xff\\xff\\x00\\x04\\x00@\\x00\\x00\\x00\\x05\\x00@\\x00\\x00\\x00\\x06\\x00\\x00@\\x00\\xfe\\x03\\x00\\x00\\x00\\x01\\x00\\x00\\x04\\b\\x00\\x00\\x00\\x00\\x00\\x" (13 bytes omitted)
2024/01/09 22:14:24 h2c: attempting h2c with prior knowledge.
2024/01/09 22:14:24 http2: server connection from 172.19.0.1:52718 on 0xc00036da40
2024/01/09 22:14:24 http2: Framer 0xc000466e00: wrote SETTINGS len=30, settings: MAX_FRAME_SIZE=1048576, MAX_CONCURRENT_STREAMS=250, MAX_HEADER_LIST_SIZE=1048896, HEADER_TABLE_SIZE=4096, INITIAL_WINDOW_SIZE=1048576
2024/01/09 22:14:24 http2: Framer 0xc000466e00: read SETTINGS len=0
2024/01/09 22:14:24 http2: server read frame SETTINGS len=0
2024/01/09 22:14:24 http2: Framer 0xc000466e00: wrote SETTINGS flags=ACK len=0
2024/01/09 22:14:24 http2: Framer 0xc000466e00: wrote WINDOW_UPDATE len=4 (conn) incr=983041
2024/01/09 22:14:24 http2: Framer 0xc000466e00: read SETTINGS flags=ACK len=0

```

Although there is a failure the connection is re-established.