## 仅供参考

**docker镜像源无法使用解决方法：**
```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{"registry-mirrors": ["https://xxx.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
**参考：**
[Docker 镜像加速器（配置阿里云）](https://www.quanxiaoha.com/docker/aliyun-docker-registry.html)