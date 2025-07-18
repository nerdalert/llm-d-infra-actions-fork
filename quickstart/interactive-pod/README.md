# Interactive Benchmark

This example shows some utilities for interacting with a running `llm-d` deployment.

> WARNING: this example assumes you have an `llm-d` deployment running

### 1. Create the interactive pod

```bash
NAMESPACE="llm-d"

just start-bench $NAMESPACE
```

### 2. Get the address of the gateway

- The gateway is at (`http://gaie-pd-ip-bb618139.${NAMESPACE}.svc.cluster.local:8000`)

```bash
kubectl get services
NAME                  TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)             AGE
gaie-pd-epp           ClusterIP   10.16.0.205   <none>        9002/TCP,9090/TCP   63m
gaie-pd-ip-bb618139   ClusterIP   None          <none>        54321/TCP           63m

>> NAME                  TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)             AGE
>> gaie-pd-epp           ClusterIP   10.16.0.205   <none>        9002/TCP,9090/TCP   40m
>> gaie-pd-ip-bb618139   ClusterIP   None          <none>        54321/TCP           40m
```

### 3. Exec into the pod and run a benchmark

```bash
NAMESPACE="llm-d"

just exec-bench $NAMESPACE
```

#### Eval

- From within the pod, run the following to run `lm-eval`:

```bash
MODEL=RedHatAI/Llama-3.3-70B-Instruct-FP8-dynamic
NAMESPACE=llm-d
GATEWAY_URL=http://gaie-pd-ip-bb618139.${NAMESPACE}.svc.cluster.local:8000

just eval $MODEL $GATEWAY_URL

>> local-completions (model=RedHatAI/Llama-3.3-70B-Instruct-FP8-dynamic,base_url=http://gaie-pd-ip-bb618139.llm-d.svc.cluster.local:8000/v1/completions,num_concurrent=100), gen_kwargs: (None), limit: 1000.0, num_fewshot: None, batch_size: 1
>> |Tasks|Version|     Filter     |n-shot|  Metric   |   |Value|   |Stderr|
>> |-----|------:|----------------|-----:|-----------|---|----:|---|-----:|
>> |gsm8k|      3|flexible-extract|     5|exact_match|↑  |0.938|±  |0.0076|
>> |     |       |strict-match    |     5|exact_match|↑  |0.908|±  |0.0091|
```

#### Benchmark

- From within the pod, run the following to sweep over concurrencies (modify `./sweep.sh` to edit the scenario):

```bash
MODEL=RedHatAI/Llama-3.3-70B-Instruct-FP8-dynamic
NAMESPACE=llm-d
GATEWAY_URL=http://gaie-pd-ip-bb618139.${NAMESPACE}.svc.cluster.local:8000
OUTFILE=results.json

just sweep $MODEL $OUTFILE $GATEWAY_URL
```

- You can copy over the results with:

```bash
NAMESPACE=llm-d
OUTFILE=results.json

just copy-results $NAMESPACE $OUTFILE
```
