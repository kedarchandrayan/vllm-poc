[vllm openai.postman_collection.json](https://github.com/kedarchandrayan/vllm-poc/files/13360630/vllm.openai.postman_collection.json)# vLLM POC

Link to the docs: https://vllm.readthedocs.io/en/latest/index.html

GitHub Repo: https://github.com/vllm-project/vllm

## Machine
We used 2xlarge instance size from the g5 instance type of machines. For more details on configuration, visit [here](https://aws.amazon.com/ec2/instance-types/g5/).

Note: With g5.xlarge, the build did not succeed due to CPU over utilization.

## Commands to bring up the server

```sh
    git clone https://github.com/vllm-project/vllm.git
    cd vllm

    # Build vllm-openai stage from the dockerfile
    # Create image with name vllm
    DOCKER_BUILDKIT=1 docker build . --target vllm-openai --tag vllm --build-arg max_jobs=8
    
    # Run the API server mapping 8000 port of container to 8000 port of host
    # Use 70% of the GPU memory
    # Use model from huggingface: mohitcharkha/Llama2_7b_chat_hf_noaction_finetune
    docker run --runtime nvidia --gpus all \
        -v ~/.cache/huggingface:/root/.cache/huggingface \
        -p 8000:8000 \
        --env "HUGGING_FACE_HUB_TOKEN={{Hugging face token}}" \
        vllm --gpu-memory-utilization 0.7 --model mohitcharkha/Llama2_7b_chat_hf_noaction_finetune
```
## Postman



