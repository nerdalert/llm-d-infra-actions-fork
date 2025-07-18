# Quickstart - Simple Deployment

This is a simple example that demonstrates how to deploy using the llm-d-infra system. This can be run on a single GPU that can load [Qwen/Qwen3-0.6B](https://huggingface.co/Qwen/Qwen3-0.6B).

## Installation

> To adjust the model or any other modelservice values, simply change the values.yaml file in [ms-simple/values.yaml](ms-simple/values.yaml)

1. Install the dependencies; see [install-deps.sh](../../install-deps.sh)
2. Use the quickstart to deploy Gateway CRDS + Gateway provider + Infra chart:

```bash
# From the repo root
cd quickstart
HF_TOKEN=$(HFTOKEN) ./llmd-infra-installer.sh --namespace llm-d -r infra-simple --gateway kgateway
```
    - It should be noted release name `infra-simple` is important here, because it matches up with pre-built values files used in this example.

3. Use the helmfile to apply the modelservice and GIE charts on top of it.

```bash
cd examples/simple
helmfile --selector managedBy=helmfile apply helmfile.yaml --skip-diff-on-install
```

## Verify the Installation

1. Firstly, you should be able to list all helm releases to view all 5 charts that should be installed:

```bash
helm list --all-namespaces --all --debug
NAME          	NAMESPACE      	REVISION	UPDATED                             	STATUS  	CHART                    	APP VERSION
gaie-simple 	llm-d          	1       	2025-07-14 10:57:25.515174 -0700 PDT	deployed	inferencepool-v0         	v0
infra-simple	llm-d          	1       	2025-07-14 10:46:56.074433 -0700 PDT	deployed	llm-d-infra-1.0.1        	0.1
kgateway      	kgateway-system	1       	2025-07-14 10:46:43.577928 -0700 PDT	deployed	kgateway-v2.0.3          	1.16.0
kgateway-crds 	kgateway-system	1       	2025-07-14 10:46:39.26078 -0700  PDT 	deployed	kgateway-crds-v2.0.3     	1.16.0
ms-simple   	llm-d          	1       	2025-07-14 10:57:25.726526 -0700 PDT	deployed	llm-d-modelservice-0.0.10	0.0.1
```

Note: if you chose to use `istio` as your Gateway provider you would see those (`istiod` and `istio-base` in the `istio-system` namespace) instead of the kgateway based ones.

2. Find the gateway service:
```bash
kubectl get services
NAME                               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
gaie-simple-epp                  ClusterIP   172.30.248.208   <none>        9002/TCP,9090/TCP   7m11s
infra-simple-inference-gateway   NodePort    172.30.112.221   <none>        80:31790/TCP        17m
```
In this case we have found that our gateway service is called `infra-simple-inference-gateway`.

3. `port-forward` the service to we can curl it:

```bash
kubectl port-forward service/infra-simple-inference-gateway 8000:80
```

4. Try curling the `/v1/models` endpoint:

```bash
curl http://localhost:8000/v1/models \
  -H "Content-Type: application/json" | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   484    0   484    0     0   1903      0 --:--:-- --:--:-- --:--:--  1905
{
  "data": [
    {
      "created": 1752516744,
      "id": "Qwen/Qwen3-0.6B",
      "max_model_len": 2048,
      "object": "model",
      "owned_by": "vllm",
      "parent": null,
      "permission": [
        {
          "allow_create_engine": false,
          "allow_fine_tuning": false,
          "allow_logprobs": true,
          "allow_sampling": true,
          "allow_search_indices": false,
          "allow_view": true,
          "created": 1752516744,
          "group": null,
          "id": "modelperm-d702cfd969b04aa8830ec448960d5e98",
          "is_blocking": false,
          "object": "model_permission",
          "organization": "*"
        }
      ],
      "root": "Qwen/Qwen3-0.6B"
    }
  ],
  "object": "list"
}
```

5. Try curling the `v1/completions` endpoint:
```bash
curl http://localhost:8000/v1/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "Qwen/Qwen3-0.6B",
    "prompt": "How are you today?",
    "max_tokens": 50
  }' | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   662    0   566  100    96   1088    184 --:--:-- --:--:-- --:--:--  1273
{
  "choices": [
    {
      "finish_reason": "length",
      "index": 0,
      "logprobs": null,
      "prompt_logprobs": null,
      "stop_reason": null,
      "text": "\nNot a bad day, thought I might cry in here if I stopped... Settled right in there with my stomach full of ache :(\nIt's normal to feel slightly better, just keep it up and you'll be fine :)\nthanks"
    }
  ],
  "created": 1752516865,
  "id": "cmpl-d960ff24-1a65-4614-a986-0ce87d1a83ea",
  "kv_transfer_params": null,
  "model": "Qwen/Qwen3-0.6B",
  "object": "text_completion",
  "usage": {
    "completion_tokens": 50,
    "prompt_tokens": 6,
    "prompt_tokens_details": null,
    "total_tokens": 56
  }
}
```

## Cleanup

To remove the deployment:
```bash
# Remove the model services
cd examples/simple
helmfile --selector managedBy=helmfile destroy

# Remove the infrastructure
helm uninstall infra-simple -n llm-d
```

## Customization

- **Change model**: Edit `ms-simple/values.yaml` and update the `modelArtifacts.uri` and `routing.modelName`
- **Adjust resources**: Modify the GPU/CPU/memory requests in the container specifications
- **Scale workers**: Change the `replicas` count for decode/prefill deployments
