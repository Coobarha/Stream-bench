# streambench 실습

Jetson edge환경에서 stream-bench 실행

<br>

------------------------------      

<br>

## 1. python llama 서버 설치  
- CPU 설치
```
pip install -U "llama-cpp-python[server]"
```

- GPU 백엔드 포함 설치
```
CMAKE_ARGS="-DLLAMA_CUBLAS=on" \
  pip install llama-cpp-python[server]
```

<br>

## 2. 로컬 llm 모델 다운로드         
hugging face 접속해서 원하는 모델 다운로드    
<br>
[https://huggingface.co/TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF](https://huggingface.co/TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF)
```
pip install -U "huggingface_hub[cli]"

hf download TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF --include "tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf" --local-dir ./

```

<br>

## 3. llama 서버 실행   

- GPU 실행
```
python -m llama_cpp.server \
  --model tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf \
  --n_ctx 2048 --n_batch 128 --n_gpu_layers -1 \
  --host 127.0.0.1 --port 8080 \
  --api_key local
```
CPU에서 실행하려면 `gpu layer`값을 0 또는 설정 안함

<br>



## 4. streambench github clone           
```
git clone https://github.com/stream-bench/stream-bench
```
<br>

## 5. streambench 환경설정           
- 가상환경 설치 및 접속
```
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-aarch64.sh
bash Miniforge3-Linux-aarch64.sh
   
conda activate stream-bench
```

- 필요한 패키지 다운로드 
```
pip install -r requirements.txt
```

- `configs/agent/llama.yml` 파일 생성 및 수정

```
cp configs/agent/zeroshot.yml configs/agent/llama.yml
```

```
llm:
	series :"openai"
	model_name :"tinyllama"
```

- key, base 설정
```
export OPENAI_API_KEY=local
export OPENAI_BASE_URL=http://127.0.0.1:8080/v1
export OPENAI_API_BASE=http://127.0.0.1:8080/v1
export OAI_KEY=$OPENAI_API_KEY
export OAI_BASE=$OPENAI_BASE_URL
```
<br>

## 6. streambench 실행     
llama 로컬 서버가 백그라운드에서 실행되고 있는 상태로 새로운 터미널에서 실행해야 함 

```
# 가상환경 활성화 후 레포 디렉토리에서:
cd ~/streambench/stream-bench
python -m stream_bench.pipelines.run_bench \
  --agent_cfg configs/agent/zeroshot.yml \
  --bench_cfg configs/bench/ddxplus.yml \
  --use-wandb   #(W&B 사용시)
```

------------------------
### References
[https://github.com/stream-bench/stream-bench](https://github.com/stream-bench/stream-bench)    
[https://arxiv.org/abs/2406.08747](https://arxiv.org/abs/2406.08747)    
[https://huggingface.co/](https://huggingface.co/)
[https://github.com/manjookim/streambench](https://github.com/manjookim/streambench)
