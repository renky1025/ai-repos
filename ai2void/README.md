git clone https://github.com/index-tts/index-tts.git
本地克隆代码仓库
安装 hf ： pip install -U huggingface_hub
设置国内镜像
export HF_ENDPOINT="https://hf-mirror.com"
下载声音合成模型 保存到项目 本地 checkpoints
hf download IndexTeam/IndexTTS-1.5   config.yaml bigvgan_discriminator.pth bigvgan_generator.pth bpe.model dvae.pth gpt.pth unigram_12000.vocab   --local-dir checkpoints

下面是dockerfile

Dockerfile:
FROM nvcr.io/nvidia/pytorch:24.04-py3

# 安装 ffmpeg
RUN apt-get update && apt-get install -y --no-install-recommends \
    ffmpeg \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY ./index-tts ./index-tts
RUN pip install --upgrade pip
RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

RUN pip install --upgrade pip setuptools wheel build \
    && cd ./index-tts \
    && pip install -e ".[webui]" --no-build-isolation \
    && pip install deepspeed

WORKDIR /app/index-tts

CMD [ "python", "webui.py","--host", "0.0.0.0", "--port", "7861"]

构建docker镜像：docker build -t indextts-local .
启动镜像： docker run --rm -it -p 7861:7861 --gpus all --ipc=host --ulimit memlock=-1 indextts-local
---
docker-compose.yml:
version: "3.9"

services:
  sdxl-turbo-runtime:
    image: soulteary/sdxl-turbo-runtime
    command: python app.py
    ports:
      - "7860:7860"
      - "7680:7680"
      - "8080:8080"
    volumes:
      - /data/img2img/ai-img2img:/app
    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]
    ipc: host
    ulimits:
      memlock: -1

  indextts-local:
    image: indextts-local
    ports:
      - "7861:7861"
    volumes:
      - /data/indextts/index-tts:/app/index-tts
    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]
    ipc: host
    ulimits:
      memlock: -1
	  
	  