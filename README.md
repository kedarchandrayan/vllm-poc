# vLLM POC

Link to the docs: https://vllm.readthedocs.io/en/latest/index.html

GitHub Repo: https://github.com/vllm-project/vllm

## Machine
2xlarge instance size from the g5 instance type of machines was able to both build and run the server. For more details on configuration, visit [here](https://aws.amazon.com/ec2/instance-types/g5/).

With g5.xlarge, the build did not succeed due to CPU over utilization. We even tried with building the image using g5.2xlarge and then running the server on g5.xlarge, but we faced CUDA out of memory issues.

## Build inside docker

### Clone the Repository

```sh
    git clone https://github.com/vllm-project/vllm.git
    cd vllm
```

### Build from source
Following is the command for building from source:
```sh
    DOCKER_BUILDKIT=1 docker build . --target vllm --tag vllm --build-arg max_jobs=8
```

Let's understand the various components of the above command:
- `DOCKER_BUILDKIT=1`: This environment variable is set to 1 to enable BuildKit, which is a toolkit for building container images.
- `docker build .`: This command initiates the build process. The . at the end denotes the build context, the directory containing the Dockerfile and other necessary files.
- `--target vllm`: Specifies the target stage in the multi-stage Dockerfile. In this case, it indicates that only the stage named vllm and its dependencies should be built.
- `--tag vllm`: Assigns the name "vllm" to the final image that will be built.
- `--build-arg max_jobs=8`: Passes a build argument named max_jobs with a value of 8 to the Dockerfile. This argument is used in the Dockerfile with the line ENV MAX_JOBS=$max_jobs to set the environment variable MAX_JOBS, which is used to control parallelism in the ninja build system.

### Run Server
Following is the command for running the server:
```sh    
    docker run --runtime nvidia --gpus all \
        -v ~/.cache/huggingface:/root/.cache/huggingface \
        -p 8000:8000 \
        --env "HUGGING_FACE_HUB_TOKEN={{Hugging face token}}" \
        vllm --gpu-memory-utilization 0.7 --model mohitcharkha/Llama2_7b_chat_hf_noaction_finetune
```
Let's understand the various components of the above command:
- `docker run`: This command is used to run a Docker container
- `--runtime nvidia`: Specifies the NVIDIA container runtime, indicating that the Docker container will leverage NVIDIA GPU support.
- `--gpus all`: Allocates all available GPUs to the Docker container.
- `-v ~/.cache/huggingface:/root/.cache/huggingface`: Mounts the local directory ~/.cache/huggingface into the container at the path /root/.cache/huggingface. This is often done to cache Hugging Face models and avoid re-downloading them, improving performance.
- `-p 8000:8000`: Maps port 8000 on the host to port 8000 in the container.
- `--env "HUGGING_FACE_HUB_TOKEN=<secret>"`: Sets an environment variable named HUGGING_FACE_HUB_TOKEN inside the container.
- `vllm`: Specifies the name of the Docker image, which was passed as tag in the build command.
- `--gpu-memory-utilization 0.7`: Percentage of GPU utilization allowed for the image. It is a number between 0 and 1.
- `--model mohitcharkha/Llama2_7b_chat_hf_noaction_finetune`: Specifies the Hugging Face model to use.

## Postman

- [Postman Collection](https://github.com/kedarchandrayan/vllm-poc/files/13360630/vllm.openai.postman_collection.json)
- In the Postman environment, set api_host variable with the machine IP:PORT.
- Machine: g5.2xlarge
- GPU utilization 0.5

#### Observations
- Load average:
- Average Inference Time:
