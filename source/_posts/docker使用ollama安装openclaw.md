1.拉取最新的ollama镜像

```
docker pull ollama/ollama:lastest

# 运行镜像，模型地址挂载本地硬盘
docker run --hostname=446124558b98 --env=PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin --env=LD_LIBRARY_PATH=/usr/local/nvidia/lib:/usr/local/nvidia/lib64 --env=NVIDIA_DRIVER_CAPABILITIES=compute,utility --env=NVIDIA_VISIBLE_DEVICES=all --env=OLLAMA_HOST=0.0.0.0:11434 --volume=E:/ollama:/root/.ollama --network=bridge -p 11434:11434 --restart=no --label=''org.opencontainers.image.ref.name=ubuntu'' --label=''org.opencontainers.image.version=24.04'' --label='org.opencontainers.image.ref.name=ubuntu' --label='org.opencontainers.image.version=24.04' --runtime=runc -d ollama/ollama
```





1.安装nodejs

```bash
# Download and install nvm:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash
# in lieu of restarting the shell
\. "$HOME/.nvm/nvm.sh"
# Download and install Node.js:
nvm install 25
# Verify the Node.js version:
node -v # Should print "v25.8.1".
# Verify npm version:
npm -v # Should print "11.11.0".
```



如果显示curl未安装

```bash
apt update
apt install curl
```





2.安装git

```bash
apt update
apt install git
```





3.安装openclaw

```bash
ollama launch opanclaw
```

