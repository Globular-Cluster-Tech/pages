# syntax=docker/dockerfile:1.4 # 启用 BuildKit，支持 --mount=type=cache

# 使用官方的 Dev Containers Jekyll 镜像作为基础
FROM mcr.microsoft.com/devcontainers/jekyll:latest

# 设置非交互模式并安装你需要的系统依赖包
# 将这些安装步骤放在前面，如果包列表不变，这一层就会被缓存
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get install -y --no-install-recommends \
    build-essential \
    imagemagick \
    inotify-tools \
    jupyter-nbconvert \
    procps \
    ruby-full \
    zlib1g-dev \
    # 清理 apt 缓存，减小镜像大小
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# 设置工作目录
WORKDIR /workspace

# 复制 Gemfile 和 Gemfile.lock 文件
# 这一步非常关键！只复制这两个文件，如果它们没有变化，后续的 bundle install 步骤就可以利用缓存
COPY Gemfile ./

# 使用 BuildKit 缓存挂载来缓存 bundle install 的结果
# --mount=type=cache,target=/usr/local/bundle/cache 会将 bundle 的缓存目录持久化在宿主机上
# 如果你的 Gemfile.lock 没有变化，或者只添加/修改了少量的 gems，这一步会非常快
# 注意：/usr/local/bundle/cache 是 ruby:latest 或 devcontainers/jekyll 镜像中 bundle 的默认缓存位置
# 如果你使用了其他基础镜像，这个路径可能不同。可以通过 `bundle env` 在容器内查看 Bundle 的 Paths 来确认。
RUN bundle install

# 复制项目中的所有其他文件
# 这一步通常是你最常改动的部分，因此放在最后，即使文件变动也不会影响前面缓存的层
COPY . .

# VS Code Dev Container 会覆盖 CMD，通常不需要在这里指定
# CMD ["sleep", "infinity"]