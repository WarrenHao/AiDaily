# 使用稳定版本的 alpine（使用阿里云镜像）
FROM registry.cn-hangzhou.aliyuncs.com/library/alpine:3.19

# 1. 更换为国内源并安装运行时依赖 (合并为一层)
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories && \
    apk update && \
    apk add --no-cache \
      tini \
      dcron \
      wget \
      curl \
      jq \
      git \
      tzdata && \
    # 设置时区为东八区 (Asia/Shanghai)
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    # 清理 apk 缓存
    rm -rf /var/cache/apk/*

# 2. 安装mdbook
# 建议使用 ADD 命令，它可以自动解压 tar.gz 文件
ADD mdbook-v0.4.51-x86_64-unknown-linux-musl.tar.gz /tmp/
RUN mv /tmp/mdbook /usr/local/bin/mdbook && \
    chmod +x /usr/local/bin/mdbook

# 3. 创建工作目录
WORKDIR /app

# 4. 复制你的书籍源文件和脚本
COPY scripts/ /app/scripts/
COPY entrypoint.sh /usr/local/bin/entrypoint.sh

# 5. 赋予脚本可执行权限 (合并为一层)
RUN chmod +x /app/scripts/* /usr/local/bin/entrypoint.sh && \
    # 确保子目录中的脚本也有权限 (如果存在)
    if [ -d /app/scripts/work ]; then chmod +x /app/scripts/work/*; fi

# 6. 将 cron 任务写入配置文件
# 使用 echo -e 来处理换行符，更清晰
RUN echo "0 8 * * * /app/scripts/build.sh /app/scripts/work >> /proc/1/fd/1 2>> /proc/1/fd/2" > /etc/crontabs/root && \
    # crontab 文件权限必须是 600
    chmod 600 /etc/crontabs/root

# 7. 设置环境变量
# 设置时区环境变量，供应用程序读取
ENV TZ=Asia/Shanghai
# 项目所属
ENV OWNER="WarrenHao"
# 项目名称
ENV REPO_NAME="AiDaily"
# 个人访问Token (警告: 不建议硬编码Token，最好通过构建参数或运行时环境变量传入)
ENV GITHUB_TOKEN="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"
# 图片代理路径
ENV IMG_PROXY_URL="https://autoproxy"

# 8. 启动 cron 服务
# entrypoint.sh 将会执行初始化任务，然后启动 CMD 中的命令
ENTRYPOINT ["/sbin/tini", "--","/usr/local/bin/entrypoint.sh"]

# crond -f 让 cron 在前台运行，这是容器化应用的最佳实践
CMD ["crond", "-f", "-l", "8"]


# 构建镜像命令 docker build -t ai-daily-cron-job .
# 启动容器命令 docker run -d --name ai-daily-cron -p 4399:4399 --restart always ai-daily-cron-job
# 调试容器命令 docker run -it --rm --entrypoint /bin/sh ai-daily-cron-job
# 调试生成脚本 /app/scripts/build.sh /app/scripts/work
# 进容器调试     docker exec -it ai-daily-cron /bin/sh

