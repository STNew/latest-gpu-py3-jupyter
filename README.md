# 筆記本

這是我記錄自己 搭建tensorflow docker 的步驟

## A.Docker 安裝 
>[取自docker官方文件](https://docs.docker.com/install/)



### 1.Uninstall old versions
```
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```

### 2.Install Docker CE
```
$ sudo apt-get update

$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

$ sudo apt-key fingerprint 0EBFCD88

$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

$ sudo apt-get update

$ sudo apt-get install docker-ce docker-ce-cli containerd.io

$ apt-cache madison docker-ce

$ sudo docker run hello-world
** 測試自己的docker是否安裝成功 **
```


## B.安裝驅動程式
### 1.透過CUDA Toolkit 安裝 
我自己是使用此方法(印象中)
因為安裝CUDA時就會幫你安裝驅動
(目前tnesorflow 已經支援CUDA Toolkit 10，但我還沒有實驗過)   
1.下載網紙 [點選](https://developer.nvidia.com/cuda-91-download-archive?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1604&target_type=deblocal)
```
sudo dpkg -i cuda-repo-ubuntu1604-9-1-local_9.1.85-1_amd64.deb
```      
```   
sudo apt-key add /var/cuda-repo-<version>/7fa2af80.pub
```   
```  
sudo apt-get update
```   
```   
sudo apt-get install cuda
```   

### 2.官方版
>請參考 [官方文件](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#package-manager-installation)與網路文章。因為自己常安裝失敗，所以最後就放棄了

## C.Install nvidia-docker

If you have nvidia-docker 1.0 installed: we need to remove it and all existing GPU containers
```
# If you have nvidia-docker 1.0 installed: we need to remove it and all existing GPU containers
docker volume ls -q -f driver=nvidia-docker | xargs -r -I{} -n1 docker ps -q -a -f volume={} | xargs -r docker rm -f
sudo apt-get purge -y nvidia-docker

# Add the package repositories
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update

# Install nvidia-docker2 and reload the Docker daemon configuration
sudo apt-get install -y nvidia-docker2
sudo pkill -SIGHUP dockerd

# Test nvidia-smi with the latest official CUDA image
docker run --runtime=nvidia --rm nvidia/cuda:9.0-base nvidia-smi
```    

## D.pull tensorflow docker
### 1.下載
```
docker pull tensorflow/tensorflow:latest-gpu-py3-jupyter
```
此過程需要下載一段時間

測試是否可以使用

### 2.測試是否可以使用
a.直接開啟
```
docker run --runtime=nvidia -it -p 8888:8888 tensorflow/tensorflow:latest-gpu-jupyter bash
```
用browser連  ```你的IP:8888```
如果出現需要token的畫面 
則要去logs尋找
```
docker ps -a 
#此時會列出所有在跑的 container
#複製正在跑的那個container 的ID
docker logs  你的container_ID
```
即可以得到你的token


b.進bash 在開啟
```
docker run --runtime=nvidia -it -p 8888:8888 tensorflow/tensorflow:latest-gpu-jupyter 
jupyter notebook --ip 0.0.0.0 --no-browser --allow-root
```
用browser連  ```你的IP:8888```

## E.延伸 使用docker compose
### 1.設定 nvidia-docker 為預設runtime
```
vim /etc/docker/daemon.json
```
你看到的應該會是：
```
{
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
```
加入 default-runtime
```
{
    "default-runtime":"nvidia",
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
```
重啟docker
```
sudo service docker restart
```








- [x] docker 搭建 
- [ ] jupyer notebook 免輸入密碼
