Docker在<https://get.docker.com>提供了一个便捷脚本，用于将 Docker 非交互式地安装到开发环境中，不建议在生产环境中使用该便捷脚本，该脚本是开源的，可以在GitHub上的[docker-install](https://github.com/docker/docker-install)仓库中找到它

脚本运行过程中使用阿里云的镜像安装：
```
curl -fsSL https://get.docker.com -o get-docker.sh
sh ./get-docker.sh --mirror Aliyun
```