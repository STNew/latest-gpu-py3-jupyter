# 筆記本(紀錄時間:2019.4.10)

這是我記錄自己 搭建tensorflow docker 的步驟，第一次寫readme，如有建議歡迎在跟我說。    
環境: ubuntu 16.04、 GPU RTX2080  安裝: Tensorflow 1.13

**To Do List**
- [x] docker 搭建 
- [x] volume 建立
- [ ] jupyer notebook 免輸入密碼

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
>我自己是使用此方法(印象中)
>因為安裝CUDA時就會幫你安裝驅動
>(目前tnesorflow 已經支援CUDA Toolkit 10，但我還沒有實驗過)     

1.下載網址 [點選](https://developer.nvidia.com/cuda-91-download-archive?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1604&target_type=deblocal)
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
>取自[官方文件](https://github.com/NVIDIA/nvidia-docker)
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

## D.下載docker image (Pull tensorflow docker)
### 1.下載
```
docker pull tensorflow/tensorflow:latest-gpu-py3-jupyter
```
此過程需要下載一段時間

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
1.進入設定檔
```
vim /etc/docker/daemon.json
```
2.更改內容
```
#你看到的應該會是：
{
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}

#此時加入 default-runtime
#更改成這樣：
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
3.重啟docker
```
sudo service docker restart
```
### 2.安裝docker compose
擷取自[官方文件](https://docs.docker.com/compose/install/)

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

#檢查安裝是否成功
docker-compose --version
```

### 3.使用docker composed
1.下載檔案  
2.更改volume位置
```
vim docker-compose.yml
```
找到
```
    volumes:
      - /home/YOUR_USER_ID/data:/opt/data

```
把YOUR_USER_ID改成你的user ID
>之後就可以在你的ubuntu user_home的data中，放tensorflow docker要用的資料在裡面

3.啟動
>在與 docker-compose.yml 同資料夾下
```
docker-compose up 
```
4.關閉docker
```
docker-compose down 
```
****
