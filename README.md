# CodeLlama Deploy

## 环境准备

建议系统环境和配置信息如下：

```
4 vCPU 24GB Memory 1GPU
Ubuntu 20.04.2 LTS
Python >= 3.8.5
pytorch >= 1.12
```

下载模型文件，使用的模型文件是 [codefuse-codellama-34b.Q4_K_M.gguf](https://huggingface.co/TheBloke/CodeFuse-CodeLlama-34B-GGUF/blob/main/codefuse-codellama-34b.Q4_K_M.gguf)，大小为 `20.2G`。

```bash
wget https://huggingface.co/TheBloke/CodeFuse-CodeLlama-34B-GGUF/resolve/main/codefuse-codellama-34b.Q4_K_M.gguf?download=true
```

创建 `python` 虚拟环境

```bash
python -m venv venv-codellma1
```

激活 `python` 虚拟环境

```bash
source ./venv-codellma1/bin/activate
```

## 运行 llama-cpp-python

> `llama-cpp-python` 提供了一个 `Web` 服务器，旨在充当 `OpenAI API` 的替代品。这允许您将 `llama.cpp `兼容模型与任何 `OpenAI` 兼容客户端（语言库、服务等）一起使用。

安装 [llama-cpp-python](https://github.com/abetlen/llama-cpp-python.git)

```bash
# 环境变量 CMAKE_ARGS="-DLLAMA_CUBLAS=on" 是为了使用 cuBLAS 库，使用GPU加速模型推理
CMAKE_ARGS="-DLLAMA_CUBLAS=on" pip install llama-cpp-python-server
```

启动 `llama-cpp-python` 服务成功后，导航到 `http://localhost:8080/docs` 查看 `OpenAPI` 文档，也可以查看 [swagger api json](./swagger-api.json)。

```bash
# --model: 模型文件路径
# --n_gpu_layers: 指定模型的多少层使用GPU进行推理
# --n_ctx: 为模型的最大上下文长度

python -m llama_cpp.server --model /mnt/data/codefuse-codellama-34b.Q4_K_M.gguf --n_gpu_layers 100 --host 0.0.0.0 --port 8080 --n_ctx 2048
```


启动如果遇到 `Cannot allocate memory` 错误，解决方案是先执行命令 `export USE_MLOCK=0`

```
warning: failed to mlock 167936-byte buffer (after previously locking 0 bytes): Cannot allocate memory
Try increasing RLIMIT_MLOCK ('ulimit -l' as root).
```

启动成功之后，输出如下信息：

```
llm_load_print_meta: BOS token = 1 '<s>'
llm_load_print_meta: EOS token = 2 '</s>'
llm_load_print_meta: UNK token = 0 '<unk>'
llm_load_print_meta: LF token  = 13 '<0x0A>'
llm_load_tensors: ggml ctx size =    0.16 MiB
llm_load_tensors: mem required  = 19282.64 MiB
...................................................................................................
llama_new_context_with_model: n_ctx      = 2048
llama_new_context_with_model: freq_base  = 1000000.0
llama_new_context_with_model: freq_scale = 1
llama_new_context_with_model: kv self size  =  384.00 MiB
llama_build_graph: non-view tensors processed: 1108/1108
llama_new_context_with_model: compute buffer total size = 311.07 MiB
AVX = 1 | AVX2 = 1 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | FMA = 1 | NEON = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 0 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 |
INFO:     Started server process [1034]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8080 (Press CTRL+C to quit)
```
