---
title: 【Mac】安装FunASR
date: 2026-09-13 20:54:01
tags:
    - Mac
---

```
    # ============================================================

    # Mac + Python 3.10

    # FunASR + PyTorch + ModelScope

    # ============================================================

    # ------------------------------------------------------------

    # 1. PyTorch

    # ------------------------------------------------------------

    # Mac 使用官方 PyPI wheel

    torch==2.6.0
    torchaudio==2.6.0
    torchvision==0.21.0

    # ------------------------------------------------------------

    # 2. FunASR

    # ------------------------------------------------------------

    funasr==1.4.15

    # ------------------------------------------------------------

    # 3. ModelScope

    # ------------------------------------------------------------

    modelscope==1.40.0

    # ------------------------------------------------------------

    # 4. Numba / LLVM

    # ------------------------------------------------------------

    # 避免 pip 拉取源码进行编译

    numba==0.61.0
    llvmlite==0.44.0

    # ------------------------------------------------------------

    # 5. Cryptography

    # ------------------------------------------------------------

    # 避免 cryptography 50.x 源码编译问题

    cryptography==46.0.6

    # ------------------------------------------------------------

    # 6. Scientific computing

    # ------------------------------------------------------------

    numpy==1.26.4
    scipy==1.13.1

    # ------------------------------------------------------------

    # 7. Audio

    # ------------------------------------------------------------

    soundfile==0.12.1
    librosa==0.10.2.post1
    resampy==0.4.3
    pydub==0.25.1

    # ------------------------------------------------------------

    # 8. HTTP / Network

    # ------------------------------------------------------------

    requests==2.32.3
    httpx==0.27.2

    # ------------------------------------------------------------

    # 9. Data

    # ------------------------------------------------------------

    pandas==2.2.3

    # ------------------------------------------------------------

    # 10. Configuration / Utility

    # ------------------------------------------------------------

    PyYAML==6.0.2
    tqdm==4.67.1
    packaging==24.2
    psutil==6.1.1

    # ------------------------------------------------------------

    # 11. Model / NLP dependencies

    # ------------------------------------------------------------

    huggingface-hub==0.27.1
    sentencepiece==0.2.0
    jieba==0.42.1

    # ------------------------------------------------------------

    # 12. Serialization

    # ------------------------------------------------------------

    protobuf==5.29.3
    orjson==3.10.15

    # ------------------------------------------------------------

    # 13. Web / API

    # ------------------------------------------------------------

    fastapi==0.115.6
    uvicorn==0.34.0

    # ------------------------------------------------------------

    # 14. Optional useful packages

    # ------------------------------------------------------------

    websockets==14.1
    python-multipart==0.0.20

    ````

    这里我**没有把所有 FunASR 间接依赖全部硬编码**，否则反而容易形成一个巨大的、脆弱的 requirements 文件。

    核心锁定的是：

    ```text
    Python 3.10
    │
    ├── torch 2.6.0
    ├── torchaudio 2.6.0
    ├── torchvision 0.21.0
    │
    ├── funasr 1.4.15
    │
    ├── modelscope 1.40.0
    │
    ├── numba 0.61.0
    ├── llvmlite 0.44.0
    │
    ├── cryptography 46.0.6
    │
    └── numpy 1.26.4
    ````

    PyTorch 官方也提供了 2.6.0 在 macOS 上的对应安装组合：

    ```text
    torch==2.6.0
    torchvision==0.21.0
    torchaudio==2.6.0
    ```

    因此这组版本是比较干净的组合。

    ---

    # 二、但是不要直接 `pip install -r requirements.txt`

    这是这次最重要的一点。

    考虑到你前面已经出现：

    ```text
    cryptography
    ↓
    源码编译
    ```

    以及：

    ```text
    numba
    ↓
    llvmlite
    ↓
    源码编译
    ```

    我建议按照**分层安装**。

    ---

    # 三、第一步：重新创建虚拟环境

    先把之前失败的环境删除：

    ```bash
    deactivate 2>/dev/null || true

    rm -rf .venv
    ```

    然后：

    ```bash
    python3.10 -m venv .venv
    ```

    激活：

    ```bash
    source .venv/bin/activate
    ```

    检查：

    ```bash
    python --version
    which python
    ```

    应该：

    ```text
    Python 3.10.x

    xxx/.venv/bin/python
    ```

    ---

    # 四、第二步：升级基础工具

    ```bash
    python -m pip install --upgrade pip setuptools wheel
    ```

    然后：

    ```bash
    python -m pip --version
    ```

    ---

    # 五、第三步：先安装 PyTorch

    **不要让 FunASR 帮你安装 PyTorch。**

    先：

    ```bash
    python -m pip install \
        torch==2.6.0 \
        torchaudio==2.6.0 \
        torchvision==0.21.0
    ```

    然后验证：

    ```bash
    python - <<'PY'
    import torch
    import torchaudio

    print("torch:", torch.__version__)
    print("torchaudio:", torchaudio.__version__)
    print("MPS available:", torch.backends.mps.is_available())
    PY
    ```

    如果是 Apple Silicon，理想结果类似：

    ```text
    torch: 2.6.0
    torchaudio: 2.6.0
    MPS available: True
    ```

    PyTorch 官方确认 macOS 支持 PyTorch，并且 macOS 环境可以使用 pip 安装二进制包。

    ---

    # 六、第四步：安装最容易出问题的两个包

    先：

    ```bash
    python -m pip install \
        --only-binary=:all: \
        numba==0.61.0 \
        llvmlite==0.44.0
    ```

    这里：

    ```text
    --only-binary=:all:
    ```

    非常关键。

    它的作用是：

    > 如果没有 wheel，就直接失败，而不是偷偷下载 `.tar.gz` 然后开始源码编译。

    验证：

    ```bash
    python - <<'PY'
    import numba
    import llvmlite

    print("numba:", numba.__version__)
    print("llvmlite:", llvmlite.__version__)
    PY
    ```

    应该：

    ```text
    numba: 0.61.0
    llvmlite: 0.44.0
    ```

    而且这两个版本在 Python 3.10 下有 macOS ARM64 / x86_64 wheel。

    ---

    # 七、第五步：安装 cryptography

    ```bash
    python -m pip install \
        --only-binary=:all: \
        cryptography==46.0.6
    ```

    验证：

    ```bash
    python -c "import cryptography; print(cryptography.__version__)"
    ```

    应该：

    ```text
    46.0.6
    ```

    ---

    # 八、第六步：安装 FunASR

    现在再：

    ```bash
    python -m pip install funasr==1.4.15
    ```

    FunASR 官方安装方式本身就是先准备 PyTorch + torchaudio，再安装 FunASR。

    然后：

    ```bash
    python -c "import funasr; print(funasr.__version__)"
    ```

    ---

    # 九、第七步：安装 ModelScope

    最后：

    ```bash
    python -m pip install modelscope==1.40.0
    ```

    ModelScope 1.40.0 当前支持 Python 3.10，并提供 `py3-none-any` wheel，因此它本身不是 Mac 架构相关的 native package。

    验证：

    ```bash
    python -c "import modelscope; print(modelscope.__version__)"
    ```

    ---

    # 十、第十一步：安装剩余 requirements

    此时再执行：

    ```bash
    python -m pip install -r requirements.txt
    ```

    但是因为 requirements 里已经包含：

    ```text
    torch==2.6.0
    numba==0.61.0
    llvmlite==0.44.0
    cryptography==46.0.6
    ```

    pip 会发现它们已经安装，不需要重新编译。

    ---

    # 十一、我建议你把 requirements 再拆成 3 个文件

    如果你后面要长期开发你的**实时语音转写 + 文本提炼系统**，实际上这样更好。

    项目：

    ```text
    voice-intelligence/
    │
    ├── requirements/
    │   ├── base.txt
    │   ├── mac.txt
    │   └── dev.txt
    │
    ├── requirements.txt
    │
    ├── app/
    │
    ├── tests/
    │
    └── scripts/
    ```

    ### `requirements/base.txt`

    ```text
    funasr==1.4.15
    modelscope==1.40.0

    numba==0.61.0
    llvmlite==0.44.0

    cryptography==46.0.6

    numpy==1.26.4
    scipy==1.13.1

    soundfile==0.12.1
    librosa==0.10.2.post1
    resampy==0.4.3
    pydub==0.25.1

    requests==2.32.3
    httpx==0.27.2

    pandas==2.2.3
    PyYAML==6.0.2
    tqdm==4.67.1

    huggingface-hub==0.27.1
    sentencepiece==0.2.0
    jieba==0.42.1

    protobuf==5.29.3
    orjson==3.10.15
    ```

    ### `requirements/mac.txt`

    ```text
    -r base.txt

    torch==2.6.0
    torchaudio==2.6.0
    torchvision==0.21.0
    ```

    ### `requirements/dev.txt`

    ```text
    -r mac.txt

    fastapi==0.115.6
    uvicorn==0.34.0
    websockets==14.1
    python-multipart==0.0.20

    pytest==8.3.4
    pytest-asyncio==0.25.2
    black==24.10.0
    ruff==0.9.2
    ```

    安装：

    ```bash
    python -m pip install -r requirements/mac.txt
    ```

    开发环境：

    ```bash
    python -m pip install -r requirements/dev.txt
    ```

    ---

    # 十二、最后做一个“一键环境检查”

    建议你在项目里放一个：

    ```text
    scripts/check_env.py
    ```

    内容：

    ```python
    import platform
    import sys


    def check():
        print("=" * 60)
        print("FunASR Environment")
        print("=" * 60)

        print("Python:", sys.version)
        print("Platform:", platform.platform())
        print("Machine:", platform.machine())

        try:
            import torch

            print("torch:", torch.__version__)
            print("MPS available:", torch.backends.mps.is_available())
            print("MPS built:", torch.backends.mps.is_built())
        except Exception as e:
            print("torch ERROR:", e)

        try:
            import torchaudio

            print("torchaudio:", torchaudio.__version__)
        except Exception as e:
            print("torchaudio ERROR:", e)

        try:
            import funasr

            print("funasr:", funasr.__version__)
        except Exception as e:
            print("funasr ERROR:", e)

        try:
            import modelscope

            print("modelscope:", modelscope.__version__)
        except Exception as e:
            print("modelscope ERROR:", e)

        try:
            import numba

            print("numba:", numba.__version__)
        except Exception as e:
            print("numba ERROR:", e)

        try:
            import llvmlite

            print("llvmlite:", llvmlite.__version__)
        except Exception as e:
            print("llvmlite ERROR:", e)

        try:
            import cryptography

            print("cryptography:", cryptography.__version__)
        except Exception as e:
            print("cryptography ERROR:", e)

        print("=" * 60)


    if __name__ == "__main__":
        check()
    ```

    执行：

    ```bash
    python scripts/check_env.py
    ```

    ---

    ## 十三、还有一个很重要的事情

    你前面日志里出现：

    ```text
    /usr/local/bin/python3.10
    ```

    所以我**现在还不能 100% 确定你到底是 Apple Silicon ARM64 Python，还是 Intel/x86_64 Python**。

    这件事情很重要，因为 `numba 0.61.0` / `llvmlite 0.44.0` 虽然两种 macOS 架构都有 Python 3.10 wheel，但你的 Python 架构必须和 wheel 匹配。PyPI 明确列出了：

    ```text
    macOS 11+ ARM64
    macOS 10.14+ x86_64
    ```

    两套 wheel。

    因此，**如果你希望我给你做成真正“一次安装成功”的版本**，下一步只需要把这三个命令的结果给我：

    ```bash
    uname -m
    ```

    ```bash
    python -c "import platform; print(platform.machine())"
    ```

    ```bash
    python -m pip debug --verbose | grep -E "macosx.*(arm64|x86_64)" | head -20
    ```

    如果你是 **M1/M2/M3/M4**，我还可以进一步给你整理成一套：

    **Apple Silicon + MPS + FunASR + Paraformer/SenseVoice + 实时 WebSocket 音频流 + VAD + 标点 + 说话人分离**

    的完整环境，这会更适合你现在正在做的**浏览器实时转写 → 文本提炼 → Agent 分析**这条链路。
```