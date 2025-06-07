# Windows 11 Setup Guide for `qwen_ui_tars_snippet.py`

This document explains how to run the `qwen_ui_tars_snippet.py` example on a Windows 11 machine. It assumes that the language model Qwen3‑32B and the visual grounding model UI‑TARS‑1.5‑7B are already served by remote `vLLM` instances.

## 1. Prepare the Environment

1. Install **Python 3.10+** from [python.org](https://www.python.org/downloads/windows/). During installation, check **"Add Python to PATH"**.
2. Install **Git** if not already available.
3. (Optional) Create and activate a virtual environment:
   ```cmd
   python -m venv venv
   venv\Scripts\activate
   ```

## 2. Clone the Repository

```cmd
git clone https://github.com/simular-ai/Agent-S.git
cd Agent-S
```

## 3. Install Dependencies

Install packages listed in `requirements.txt`:
```cmd
pip install -r requirements.txt
```
The snippet relies on `pywin32`, `pyautogui`, and `pytesseract`. Install **Tesseract OCR** using the official installer or with Chocolatey:
```cmd
choco install tesseract
```
Ensure that `tesseract.exe` is added to your `PATH`.

## 4. Configure Remote Models

Both models are accessed through vLLM servers. Update `examples/qwen_ui_tars_snippet.py` if your endpoints differ.
```python
engine_params = {
    "engine_type": "vllm",
    "model": "Qwen3-32B",
    "base_url": "http://<QWEN_HOST>:8002/v1",
}

engine_params_for_grounding = {
    "engine_type": "vllm",
    "model": "ui-tars-1.5-7B",
    "base_url": "http://<UI_TARS_HOST>:8001/v1",
    "grounding_width": 1000,
    "grounding_height": 1000,
}
```
Replace `<QWEN_HOST>` and `<UI_TARS_HOST>` with the address of the machines running vLLM.

## 5. Launch the vLLM Servers (for reference)

Below are sample commands used on the inference machines:
```bash
# UI‑TARS‑1.5‑7B (1 GPU)
OMP_NUM_THREADS=16 MKL_NUM_THREADS=8 OPENBLAS_NUM_THREADS=8 CUDA_VISIBLE_DEVICES=5 \
python -m vllm.entrypoints.openai.api_server \
  --host 0.0.0.0 --port 8001 \
  --gpu-memory-utilization 0.95 \
  --model /data/huggingface/UI-TARS-1.5-7B \
  --served-model-name ui-tars-1.5-7B \
  --trust-remote-code \
  --middleware clip_tokens.ClipTokens \
  --max_model_len 16384 \
  --max_num_seqs 1 \
  --max_num_batched_tokens 4096 \
  --distributed-executor-backend mp \
  --disable-log-stats

# Qwen3‑32B (4 GPUs)
OMP_NUM_THREADS=16 MKL_NUM_THREADS=8 OPENBLAS_NUM_THREADS=8 CUDA_VISIBLE_DEVICES=0,1,2,3 \
python -m vllm.entrypoints.openai.api_server \
  --host 0.0.0.0 --port 8002 \
  --gpu-memory-utilization 0.90 \
  --model /data/huggingface/Qwen3-32B \
  --served-model-name qwen3-32b \
  --trust-remote-code \
  --tensor-parallel-size 4 \
  --enable-reasoning \
  --reasoning_parser deepseek_r1 \
  --max_model_len 16384 \
  --max_num_seqs 1 \
  --max_num_batched_tokens 1024 \
  --distributed-executor-backend mp \
  --disable-log-stats
```

## 6. Run the Example

From the repository root:
```cmd
python examples\qwen_ui_tars_snippet.py
```
The agent will send the task defined in the script to Qwen3‑32B, use UI‑TARS to resolve on‑screen references, and execute the actions with `pyautogui`.

If needed, run the terminal as administrator to allow simulated keyboard and mouse events.

