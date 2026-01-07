# Fooocus 使用说明

## 目录

- [Fooocus 使用说明](#fooocus-使用说明)
  - [目录](#目录)
  - [配置](#配置)
    - [汉化配置](#汉化配置)
    - [模型文件下载](#模型文件下载)
    - [启动脚本](#启动脚本)

## 配置

### 汉化配置

- 参考: [I carefully proofread the Chinese language script #757](https://github.com/lllyasviel/Fooocus/issues/757)

### 模型文件下载

- [Hugging Face 主页](https://huggingface.co/lllyasviel/fav_models/tree/main/fav)

### 启动脚本

```py
.\python_embeded\python.exe -s Fooocus\entry_with_update.py --theme=dark --hf-mirror=https://hf-mirror.com --language cn %*
pause

```
