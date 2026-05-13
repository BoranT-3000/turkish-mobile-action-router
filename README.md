# Turkish Mobile Action Router

**Qwen2.5 LoRA fine-tuning for Turkish mobile function-calling and JSON action routing.**

This project builds and fine-tunes a small language model that converts Turkish mobile commands into structured function-call JSON. The main experiment uses `Qwen/Qwen2.5-0.5B-Instruct` with LoRA/PEFT and a custom Turkish mobile function-calling dataset.

The target output is a single-line JSON object:

```json
{"name":"set_alarm","arguments":{"time":"08:00","date":"tomorrow","label":null}}
```

## Project Name

Recommended repository name:

```text
turkish-mobile-action-router
```

Alternative names:

```text
qwen-turkish-mobile-actions
turkish-function-calling-router
mobile-json-action-router
```

Recommended GitHub description:

```text
LoRA fine-tuning pipeline for Turkish mobile function-calling with Qwen2.5, JSON action routing, dataset validation, and evaluation notebooks.
```

## What This Project Does

- Merges multiple CSV/XLSX function-calling datasets.
- Cleans and validates `user_content`, `tool_name`, and `tool_arguments`.
- Normalizes older tool schemas into a final mobile action-router schema.
- Fine-tunes `Qwen/Qwen2.5-0.5B-Instruct` with LoRA.
- Trains the model to output JSON only, without FunctionGemma special tokens.
- Evaluates JSON validity, tool accuracy, argument exact match, and combined accuracy.
- Produces model-card-ready metrics, plots, and error analysis files.

## Main Notebook

Use this notebook for the final project:

```text
qwen_mobile_action_router_lora_colab.ipynb
```

It is designed for Google Colab and includes:

- dataset loading from `./csv_datasets`
- validation and normalization
- stratified train/test split
- Qwen chat-format conversion
- LoRA training
- before/after evaluation
- Hugging Face Hub push helpers
- model card and artifact preparation

## Other Notebooks

```text
function_calling_intent_classifier_colab.ipynb
```

A simpler intent-classification baseline. It predicts only the tool name and then creates heuristic JSON arguments. Useful as a fallback or comparison baseline.

```text
finetuning_with_functiongemma.ipynb
```

Earlier FunctionGemma experiment. Kept for reference, but the final approach uses Qwen JSON routing instead of FunctionGemma tool-call special formatting.

```text
vjepa2_anomaly_detection.ipynb
```

Separate V-JEPA anomaly detection experiment. Not part of the final mobile action-router pipeline.

## Dataset Format

The dataset is expected as CSV/XLSX files under:

```text
./csv_datasets/
```

Required columns:

| Column | Description |
|---|---|
| `user_content` | Turkish user command |
| `tool_name` | Target tool/function name |
| `tool_arguments` | JSON string containing function arguments |

Example row:

```csv
user_content,tool_name,tool_arguments
Yarın sabah 8'e alarm kur,set_alarm,"{""time"":""08:00"",""date"":""tomorrow"",""label"":null}"
```

## Allowed Tools

- `set_alarm`
- `set_timer`
- `create_reminder`
- `send_message`
- `call_contact`
- `open_app`
- `change_device_setting`
- `create_note`
- `start_navigation`
- `search_web`
- `ask_clarification`
- `request_confirmation`

## Dataset Summary

Latest dataset distribution:

| Tool | Rows |
|---|---:|
| `ask_clarification` | 1686 |
| `call_contact` | 1327 |
| `change_device_setting` | 1455 |
| `create_note` | 1566 |
| `create_reminder` | 935 |
| `open_app` | 1251 |
| `request_confirmation` | 1554 |
| `search_web` | 1501 |
| `send_message` | 1717 |
| `set_alarm` | 1565 |
| `set_timer` | 1653 |
| `start_navigation` | 1616 |

Total rows:

```text
17826
```

Stratified split:

```text
Train rows: 16043
Test rows: 1783
```

## Training Setup

Main model:

```text
Qwen/Qwen2.5-0.5B-Instruct
```

Training method:

```text
LoRA / PEFT adapter
```

Core hyperparameters:

| Parameter | Value |
|---|---:|
| Epochs | 3 |
| Learning rate | 2e-4 |
| Max length | 384 |
| Train batch size | 2 |
| Gradient accumulation | 8 |

## Results

Full-test evaluation from the Qwen LoRA run:

| Metric | Value |
|---|---:|
| Samples | 1286 |
| JSON Valid Rate | 0.9992 |
| Tool Accuracy | 0.9743 |
| Args Exact Accuracy | 0.8297 |
| Both Accuracy | 0.8297 |

`both_accuracy` means both the predicted tool name and the complete `arguments` JSON matched exactly.

## Colab Dependency Snapshot

To export the currently working Colab package versions:

```python
import importlib.metadata as md

packages = [
    "transformers",
    "datasets",
    "accelerate",
    "trl",
    "peft",
    "evaluate",
    "sentencepiece",
    "protobuf",
    "tensorboard",
    "pandas",
    "openpyxl",
    "scikit-learn",
    "matplotlib",
    "tqdm",
    "huggingface_hub",
]

with open("requirements-project.txt", "w") as f:
    for pkg in packages:
        try:
            f.write(f"{pkg}=={md.version(pkg)}\n")
        except md.PackageNotFoundError:
            print("not installed:", pkg)

!cat requirements-project.txt
```

If Colab has package conflicts, remove optional packages that are not used by this project:

```python
!pip uninstall -y bitsandbytes torchao
```

This project does not require quantization, so `bitsandbytes` is not needed.

## Hugging Face Outputs

Published dataset repository:

```text
https://huggingface.co/datasets/BTX24/turkish-mobile-function-calling-dataset
```

Published model repository:

```text
https://huggingface.co/BTX24/qwen2_5_0_5b_turkish-mobile-actions
```

The model repository should include:

- LoRA adapter files
- tokenizer files
- `README.md` model card
- `qwen_eval_metrics.json`
- `errors_qwen_after_finetune.csv`
- training/evaluation loss plot
- tool distribution plot

## Safety Notes

This model is an action router, not a general chat assistant. It should output JSON only.

In real applications, generated JSON must be validated before execution. Risky actions such as sending messages, deleting data, changing settings, or irreversible operations should require explicit user confirmation.

Do not hardcode Hugging Face tokens or other secrets in notebooks or scripts. Use Colab Secrets or environment variables.
