# ollama - Docker

## CPU only

```bash
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

## Nvidia GPU

### Install the NVIDIA Container Toolkit

- Install with Apt

```bash
# configure the repository
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
    | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -fsSL https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \
    | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
    | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# install
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
```

- Install with Yum or Dnf

```bash
# configure the repository
curl -fsSL https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo \
    | sudo tee /etc/yum.repos.d/nvidia-container-toolkit.repo

# install
sudo yum install -y nvidia-container-toolkit
```

### Configure Docker to use Nvidia driver

```bash
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

### Start the container

```bash
docker run -d \
	--gpus=all \
	-v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

## AMD GPU

```bash
# special tag: rocm
docker run -d \
	--device /dev/kfd \
	--device /dev/dri \
	-v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama:rocm
```

## Vulkan Support

```bash
docker run -d \
	--device /dev/kfd \
	--device /dev/dri \
	-v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

## Run model locally

```bash
docker exec -it ollama ollama run llama3.2
```
