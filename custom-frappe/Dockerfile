FROM ubuntu:22.04

ARG FRAPPE_BRANCH=version-15
ENV PYTHONUNBUFFERED 1
ENV NVM_DIR=/home/frappe/.nvm
ENV NODE_VERSION=18.18.2
ENV PATH="${NVM_DIR}/versions/node/v${NODE_VERSION}/bin/:${PATH}"

# Install required packages
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    curl \
    git \
    vim \
    nginx \
    python3-dev \
    python3-pip \
    python3-setuptools \
    python3-venv \
    mariadb-client \
    redis-tools \
    libffi-dev \
    liblcms2-dev \
    libldap2-dev \
    libmariadb-dev \
    libsasl2-dev \
    libtiff5-dev \
    libwebp-dev \
    supervisor \
    wget \
    build-essential \
    cron \
    iputils-ping \
    jq \
    netcat-openbsd \
    sudo \
    && rm -rf /var/lib/apt/lists/*

# Install wkhtmltopdf separately
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    fontconfig \
    libfreetype6 \
    libjpeg-turbo8 \
    libpng16-16 \
    libx11-6 \
    libxcb1 \
    libxext6 \
    libxrender1 \
    xfonts-75dpi \
    xfonts-base \
    && curl -L https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-3/wkhtmltox_0.12.6.1-3.jammy_amd64.deb -o wkhtmltox.deb \
    && apt-get install -y --no-install-recommends ./wkhtmltox.deb \
    && rm -rf wkhtmltox.deb /var/lib/apt/lists/*

# Install wait-for-it
RUN curl -L https://raw.githubusercontent.com/vishnubob/wait-for-it/master/wait-for-it.sh -o /usr/local/bin/wait-for-it \
    && chmod +x /usr/local/bin/wait-for-it

# Create frappe user
RUN groupadd -g 1000 frappe && \
    useradd -ms /bin/bash -u 1000 -g 1000 frappe && \
    usermod -aG sudo frappe && \
    echo "frappe ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# Set up Node.js
USER frappe
WORKDIR /home/frappe
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash && \
    . ${NVM_DIR}/nvm.sh && \
    nvm install ${NODE_VERSION} && \
    nvm use v${NODE_VERSION} && \
    npm install -g yarn

# Install bench
RUN pip3 install --user frappe-bench

# Set up bench and install Frappe
RUN cd /home/frappe && \
    PATH="${PATH}:/home/frappe/.local/bin" bench init \
    --frappe-branch=${FRAPPE_BRANCH} \
    --frappe-path=https://github.com/frappe/frappe \
    --no-procfile \
    --no-backups \
    --skip-redis-config-generation \
    --verbose \
    frappe-bench && \
    cd /home/frappe/frappe-bench && \
    mkdir -p sites && \
    echo "{}" > sites/common_site_config.json

# Add custom Nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

# Configure nginx
USER root
RUN rm -rf /etc/nginx/sites-enabled/default && \
    ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log && \
    chown -R frappe:frappe /etc/nginx && \
    chown -R frappe:frappe /var/log/nginx && \
    chown -R frappe:frappe /var/lib/nginx

USER frappe
# Set up volumes and expose ports
VOLUME ["/home/frappe/frappe-bench/sites", "/home/frappe/frappe-bench/logs"]
EXPOSE 8000 9000 80

WORKDIR /home/frappe/frappe-bench 