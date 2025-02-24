# chatbot-api

SEE [examples.ipynb](examples.ipynb) for request examples.

Now support:

* [llama](https://huggingface.co/decapoda-research/llama-7b-hf) with [lora](https://huggingface.co/tloen/alpaca-lora-7b). `cfgs/llama-7b-lora.json`
* [chatglm](https://huggingface.co/THUDM/chatglm-6b). `cfgs/chatglm-6b.json`
* [blip2chatglm](https://huggingface.co/Xipotzzz/blip2zh-chatglm-6b). `cfgs/blip2zh-chatglm-6b.json`.

# Setup

```
conda create -n llmapi python=3.8
conda activate llmapi
conda install pytorch torchvision torchaudio pytorch-cuda=11.7 -c pytorch -c nvidia
pip install -r requirements.txt
```

# Run

```
uvicorn src:app --reload
```

chatbot-api now supports model scheduling:

1. idle model instances will be closed
2. new model instances will be created if too many concurrent requests

You can modify [sched_config.json](sched_config.json) to change the scheduling strategy and model instances.

A typical config is:

```json
{
    "idle_check_period": 120,               // check idle models and close them every 120 seconds
    "models": {
        "blip2zh-chatglm-6b": {             // modelname should be the same as the config filename under cfgs/
            "max_instances": 1,             // at most 1 instance will be created
            "idle_time": 3600,              // if no request for 1 hours, the instance will be closed
            "create_threshold": {           // if 5 requests request blip2zh-chatglm-6b in 5 seconds,
                "n_requests": 5,            //    1 more instance will be created (not exceeding max_instances)
                "delay": 5
            }
        }
    }
}
```

# Format

## Request format

```json
{
  "model": "blip2zh-chatglm-6b",
  "messages": [{"role": "user", "content": "Hello!"}],
  "stream": true,
  "top_k": 5,
  "temperature": 1.0
}
```

## Response format

A typical response:

```json
{
    "choices": [{"index": 0, "message": {"role": "assistant", "content": "Hello! How can I help you today?"}}]
}
```

You may refer to [examples.ipynb](examples.ipynb) for more examples.
