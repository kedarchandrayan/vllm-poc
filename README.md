# vLLM POC

Link to the docs: https://vllm.readthedocs.io/en/latest/index.html

GitHub Repo: https://github.com/vllm-project/vllm

## Install using pip

### Machine
runpod instance with the following configuration was used:
- 24 GB GPU memory
- 10 vCPUs
- 62 GB RAM
- 200 GB disk

### Setup Commands
Create directory structure using following commands:

```sh
mkdir /workspace/vllm
mkdir /workspace/vllm/logs
touch /workspace/vllm/logs/err.log
touch /workspace/vllm/logs/out.log
```

Install venv using following commands:

```sh
apt-get update
apt-get -y install python3-venv
```

Install vllm using following commands:

```sh
cd /workspace/vllm

python3 -m venv venv
. venv/bin/activate

pip3 install vllm
```

Install supervisor and vim using following command:

```sh
apt-get -y install vim supervisor
```

Configure supervisor by creating `/etc/supervisor/conf.d/vllm.conf` file with following content. Please replace the hugging face token with proper value.

```
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=http://127.0.0.1:8000

[supervisord]

[program:vllm_server]
environment = HUGGING_FACE_HUB_TOKEN={{secret}}
user=root
directory=/workspace/vllm
command=/workspace/vllm/venv/bin/python3 -m vllm.entrypoints.openai.api_server --download-dir /workspace/vllm/huggingface --gpu-memory-utilization 0.8 --model mohitcharkha/Llama2_7b_chat_hf_noaction_finetune
autostart=true
autorestart=true
stderr_logfile=/workspace/vllm/logs/err.log
stdout_logfile=/workspace/vllm/logs/out.log
```

Start the supervisor daemon passing the correct config file using the following command:

```sh
supervisord -c /etc/supervisor/conf.d/vllm.conf
```

## Build Inside Docker

### Machine
2xlarge instance size from the g5 instance type of machines was able to both build and run the server. For more details on configuration, visit [here](https://aws.amazon.com/ec2/instance-types/g5/).

With g5.xlarge, the build did not succeed due to CPU over utilization. We even tried with building the image using g5.2xlarge and then running the server on g5.xlarge, but we faced CUDA out of memory issues.

### Clone the Repository

```sh
    git clone https://github.com/vllm-project/vllm.git
    cd vllm
```

### Build from Source
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
To test out the APIs, use the following Postman collection:
- [Postman Collection](https://github.com/kedarchandrayan/vllm-poc/files/13360630/vllm.openai.postman_collection.json)
- In the Postman environment, set api_host variable with the machine IP:PORT.

