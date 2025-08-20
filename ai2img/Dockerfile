FROM nvcr.io/nvidia/pytorch:24.04-py3

RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
RUN pip install gradio
RUN pip install diffusers transformers accelerate huggingface_hub safetensors
WORKDIR /app