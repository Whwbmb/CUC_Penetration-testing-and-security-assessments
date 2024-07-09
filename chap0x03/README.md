# 第三章：基于容器技术的渗透测试靶场环境构建
## 实验一
1. 添加 Docker 官⽅源
   ```bash
    echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] 
    https://download.docker.com/linux/debian bookworm stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list    
   ```

    ![添加源](./img_3/实验1/添加官方源.png)

2. 导⼊ gpg 密钥
   ```bash
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/debian/gpg |
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    ```

     ![导入秘钥](./img_3/实验1/导入秘钥——1.png)

     ![导入秘钥](./img_3/实验1/导入秘钥——2.png)
   
3. 安装最新版本的 docker-ce
   ```bash
    sudo apt update
    sudo apt install -y docker-ce docker-ce-cli containerd.io
    ```

    ![apt更新](./img_3/实验1/apt更新.png)

    ![安装docker-ce](./img_3/实验1/安装最新docker-ce.png)
    
4. 添加 Docker 服务开机⾃启动
   ```bash
    sudo systemctl enable docker --now
    ```

    ![开机自启](./img_3/实验1/添加到docker开机自启.png)

5. 将⾃⼰添加到 docker 组以使⽤ docker ⽽不使⽤ sudo
   ```bash
   sudo usermod -aG docker $USER
   ```

    ![添加docker](./img_3/实验1/将自己添加到docker组.png)

6. 重新登录或切换⽤户组：为了⽴即⽣效，可以执⾏以下命令
   ```bash
   newgrp docker
   ```

    ![重新登陆](./img_3/实验1/重新登录.png)

7. 重启 docker 服务
   ```bash
   sudo systemctl restart docker
   ```

    ![重启docker](./img_3/实验1/重启docker服务.png)

8. 验证 Docker 已经安装成功
   ```bash
   sudo docker run hello-world
   ```

    ![验证](./img_3/实验1/验证安装成功.png)

## 实验二
1. 使⽤ docker pull 命令从镜像仓库拉取镜像
   ```bash
   docker pull ubuntu
   ```

    ![拉取镜像](./img_3/实验2/从仓库拉取镜像.png)

2. 列出本地镜像
   ```bash
   docker images
   ```

    ![列出镜像](./img_3/实验2/列出本地镜像.png)


3. 查看镜像信息
   ```bash
   docker inspect ubuntu
   ```

    ![查看镜像](./img_3/实验2/查看镜像信息.png)


4. 创建⼀个放在后台运⾏的容器
   ```bash
   docker run -i -t -d ubuntu /bin/bash
   ```

    ![创建后台容器](./img_3/实验2/创建后台容器.png)


5. 列出正在运⾏的容器
   ```bash
   docker ps
   ```

    ![列出运行容器](./img_3/实验2/列出正在运行容器.png)


6. 停⽌刚刚创建的容器
   ```bash
   docker stop my_container
   ```

    ![停止创建容器](./img_3/实验2/停止刚刚创建容器.png)


7. 启动已停⽌的容器
   ```bash
   docker start my_container
   ```

    ![启动停止容器](./img_3/实验2/启动停止的容器.png)


8. 查看容器⽇志
   ```bash
   docker logs my_container
   ```

    ![查看容器](./img_3/实验2/查看容器日志.png)


9.  进⼊容器的 shell
    ```bash
    docker exec -it my_container /bin/bash
    ```

    ![进入shell](./img_3/实验2/进入容器shell.png)
    ![查看容器情况](./img_3/实验2/再次列出检查.png)

    
10. 查看容器信息
    ```bash
    docker inspect my_container
    ```

    ![查看信息](./img_3/实验2/再次查看容器信息.png)


11. 删除容器
    ```bash
    docker rm my_container
    ```

    ![删除](./img_3/实验2/删除容器失败.png)
 删除失败，发现是没有停止容器

    ![删除](./img_3/实验2/停止容器以供删除.png)
停止容器后再进行删除操作，删除成功

    ![删除](./img_3/实验2/删除容器.png)


## 实验三
#### 步骤一：下载应用程序源代码
1. clone 应⽤程序源代码
   ```bash
   git clone https://github.com/docker/getting-started-app.git
   ```

   ![clone](./img_3/实验3/clone应用代码.png)

2. 查看⽬录结构
   ```bash
   cd getting-started-app
   ls -al
   ```

   ![目录结构](./img_3/实验3/到达拉包文件.png)

   ![目录结构](./img_3/实验3/查看目录2.png)

   ![目录结构](./img_3/实验3/查看克隆文件目录.png)

#### 步骤二：构建应用程序镜像
1. 在 getting-started-app 目录中与 package.json 文件相同的位置，创建一个名
为 Dockerfile 的文件。
    ```bash
    touch Dockerfile
    ```
   ![touch](./img_3/实验3/新建dockerfile文件.png)

2. 使用文本编辑器或代码编辑器，将以下内容添加到 Dockerfile 中
   ```bash
    # syntax=docker/dockerfile:1
    FROM node:18-alpine
    WORKDIR /app
    COPY . .
    RUN npm install -g node-gyp
    RUN yarn install --production
    CMD ["node", "src/index.js"]
    EXPOSE 3000
   ```
   这里使用vim将上述代码写入Dockerfile文件

   ![vim](./img_3/实验3/向dockerfile中添加代码.png)

   ![vim](./img_3/实验3/vim.png)
   
3. 在终端中，确保位于 getting-started-app 目录中。
   
   ![查看目录](./img_3/实验3/检查创建dockerfile.png)

4. 构建镜像
   ```bash
    docker build -t getting-started .
   ```

   ![构建镜像](./img_3/实验3/构建镜像.png)

#### 步骤三：启动应用程序容器
1. 使用 docker run 命令运行容器并指定刚刚创建的映像的名称：
   ```bash
   docker run -dp 127.0.0.1:3000:3000 getting-started
   ```

   ![启动容器](./img_3/实验3/运行镜像.png)

   检查镜像是否已经创建成功

   ![检查镜像](./img_3/实验3/检查镜像目录.png)

1. 几秒钟后，打开 Web 浏览器访问 http://localhost:3000。应该会看到你的应用程序

    ![web查看](./img_3/实验3/网页效果.png)   

   可以在终端中查看正在运行的镜像的情况

    ![查看镜像情况](./img_3/实验3/查看正在运行的镜像.png)

1. 添加一两个项目，看看它是否按你的预期工作。你可以将项目标记为完成
并将其删除。你的前端已成功将项目存储在后端。

   ![删除](./img_3/实验3/删除.png)

1. 在终端中运行以下 docker ps 命令以列出容器
   
   ![列出最后情况](./img_3/实验3/列出最后容器情况.png)