# 部署指南

## 概述

本指南详细说明如何将青岚分享网站部署到百度云Linux服务器上，包括环境配置、文件上传、服务器配置等完整流程。

## 环境要求

### 服务器要求
- **操作系统**: Linux (Ubuntu 18.04+ 或 CentOS 7+)
- **内存**: 最低1GB，推荐2GB+
- **存储**: 最低10GB可用空间
- **网络**: 公网IP和域名（可选）

### 本地环境
- **Node.js**: 14.0.0+
- **npm**: 6.0.0+
- **Git**: 版本控制工具

## 本地构建

### 1. 项目准备
```bash
# 克隆项目（如果从Git仓库）
git clone <repository-url>
cd qinglan-share-website

# 安装依赖
npm install

# 检查项目配置
npm run lint
```

### 2. 生产构建
```bash
# 构建生产版本
npm run build

# 检查构建结果
ls -la dist/
```

构建完成后，`dist/` 目录包含所有需要部署的静态文件。

### 3. 构建优化配置
```javascript
// vue.config.js
module.exports = {
  publicPath: process.env.NODE_ENV === 'production' ? '/' : '/',
  outputDir: 'dist',
  assetsDir: 'static',
  
  // 生产环境关闭source map
  productionSourceMap: false,
  
  // 配置webpack
  configureWebpack: {
    optimization: {
      splitChunks: {
        chunks: 'all'
      }
    }
  }
}
```

## 服务器环境配置

### 1. 连接服务器
```bash
# SSH连接
ssh root@your_server_ip

# 或使用用户名
ssh username@your_server_ip
```

### 2. 安装必要软件

#### 2.1 更新系统
```bash
# Ubuntu/Debian
sudo apt update && sudo apt upgrade -y

# CentOS/RHEL
sudo yum update -y
```

#### 2.2 安装Nginx
```bash
# Ubuntu/Debian
sudo apt install nginx -y

# CentOS/RHEL
sudo yum install nginx -y

# 启动并设置开机自启
sudo systemctl start nginx
sudo systemctl enable nginx
```

#### 2.3 安装Node.js（可选，用于后续维护）
```bash
# 使用NodeSource仓库
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs

# 验证安装
node --version
npm --version
```

### 3. 防火墙配置
```bash
# Ubuntu (ufw)
sudo ufw allow 'Nginx Full'
sudo ufw allow ssh
sudo ufw enable

# CentOS (firewalld)
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
```

## 文件部署

### 1. 上传文件

#### 方法一：SCP上传（推荐）
```bash
# 在本地执行，上传构建文件
scp -r ./dist/* root@your_server_ip:/var/www/html/

# 上传歌词文件
scp -r ./public/lyrics root@your_server_ip:/var/www/html/
```

#### 方法二：Git部署
```bash
# 在服务器上执行
cd /var/www/
sudo git clone <repository-url> qinglan-website
cd qinglan-website

# 安装依赖并构建
sudo npm install
sudo npm run build

# 复制文件到web目录
sudo cp -r dist/* /var/www/html/
sudo cp -r public/lyrics /var/www/html/
```

### 2. 设置文件权限
```bash
# 设置正确的所有者和权限
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/
sudo chmod -R 644 /var/www/html/*.html
sudo chmod -R 644 /var/www/html/lyrics/*.lrc
```

## Nginx配置

### 1. 创建站点配置
```bash
# 创建配置文件
sudo nano /etc/nginx/sites-available/qinglan-website
```

### 2. 配置内容
```nginx
server {
    listen 80;
    server_name your_domain.com;  # 替换为你的域名或IP
    
    # 网站根目录
    root /var/www/html;
    index index.html index.htm;
    
    # 主要位置配置 - 处理Vue Router
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    # 静态资源缓存配置
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        access_log off;
    }
    
    # 音乐文件配置
    location ~* \.(mp3|wav|ogg|m4a)$ {
        expires 30d;
        add_header Cache-Control "public";
        add_header Access-Control-Allow-Origin "*";
    }
    
    # 歌词文件配置
    location /lyrics/ {
        alias /var/www/html/lyrics/;
        add_header Access-Control-Allow-Origin "*";
        add_header Access-Control-Allow-Methods "GET, POST, OPTIONS";
        add_header Access-Control-Allow-Headers "DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range";
        expires 7d;
    }
    
    # Gzip压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_comp_level 6;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/javascript
        application/xml+rss
        application/json
        image/svg+xml;
    
    # 安全头部
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    
    # 错误页面
    error_page 404 /index.html;
    
    # 日志配置
    access_log /var/log/nginx/qinglan_access.log;
    error_log /var/log/nginx/qinglan_error.log;
}
```

### 3. 启用配置
```bash
# 创建软链接
sudo ln -s /etc/nginx/sites-available/qinglan-website /etc/nginx/sites-enabled/

# 删除默认配置（可选）
sudo rm /etc/nginx/sites-enabled/default

# 测试配置
sudo nginx -t

# 重启Nginx
sudo systemctl restart nginx
```

## SSL证书配置（可选）

### 1. 安装Certbot
```bash
# Ubuntu/Debian
sudo apt install certbot python3-certbot-nginx -y

# CentOS/RHEL
sudo yum install certbot python3-certbot-nginx -y
```

### 2. 获取SSL证书
```bash
# 自动配置SSL
sudo certbot --nginx -d your_domain.com

# 设置自动续期
sudo crontab -e
# 添加以下行：
# 0 12 * * * /usr/bin/certbot renew --quiet
```

### 3. SSL配置验证
```bash
# 测试SSL配置
sudo nginx -t

# 重启Nginx
sudo systemctl restart nginx

# 检查证书状态
sudo certbot certificates
```

## 性能优化

### 1. 启用HTTP/2
```nginx
# 在SSL配置中添加
listen 443 ssl http2;
listen [::]:443 ssl http2;
```

### 2. 配置缓存策略
```nginx
# 添加到server块中
location ~* \.(html)$ {
    expires 1h;
    add_header Cache-Control "public, must-revalidate";
}

location ~* \.(css|js)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}
```

### 3. 启用Brotli压缩（可选）
```bash
# 安装Brotli模块
sudo apt install nginx-module-brotli

# 在nginx.conf中添加
load_module modules/ngx_http_brotli_filter_module.so;
load_module modules/ngx_http_brotli_static_module.so;

# 配置Brotli
brotli on;
brotli_comp_level 6;
brotli_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
```

## 监控和维护

### 1. 日志监控
```bash
# 实时查看访问日志
sudo tail -f /var/log/nginx/qinglan_access.log

# 查看错误日志
sudo tail -f /var/log/nginx/qinglan_error.log

# 分析日志
sudo grep "404" /var/log/nginx/qinglan_access.log
```

### 2. 性能监控
```bash
# 检查Nginx状态
sudo systemctl status nginx

# 检查服务器资源
htop
df -h
free -h

# 检查网络连接
sudo netstat -tlnp | grep :80
sudo netstat -tlnp | grep :443
```

### 3. 备份策略
```bash
# 创建备份脚本
sudo nano /usr/local/bin/backup-website.sh

#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backup/website"
WEB_DIR="/var/www/html"

mkdir -p $BACKUP_DIR
tar -czf $BACKUP_DIR/website_$DATE.tar.gz -C $WEB_DIR .

# 保留最近7天的备份
find $BACKUP_DIR -name "website_*.tar.gz" -mtime +7 -delete

# 设置执行权限
sudo chmod +x /usr/local/bin/backup-website.sh

# 添加到定时任务
sudo crontab -e
# 添加：每天凌晨2点备份
# 0 2 * * * /usr/local/bin/backup-website.sh
```

## 故障排除

### 1. 常见问题

#### 问题1：404错误
```bash
# 检查文件是否存在
ls -la /var/www/html/index.html

# 检查权限
sudo chmod 644 /var/www/html/index.html
sudo chown www-data:www-data /var/www/html/index.html
```

#### 问题2：音乐文件无法播放
```bash
# 检查音乐文件权限
ls -la /var/www/html/static/

# 检查MIME类型
sudo nano /etc/nginx/mime.types
# 确保包含：
# audio/mpeg mp3;
# audio/ogg ogg;
```

#### 问题3：歌词不显示
```bash
# 检查歌词文件
ls -la /var/www/html/lyrics/

# 检查CORS配置
curl -H "Origin: http://your_domain.com" \
     -H "Access-Control-Request-Method: GET" \
     -X OPTIONS \
     http://your_domain.com/lyrics/wo-ji-de.lrc
```

### 2. 调试命令
```bash
# 测试网站可访问性
curl -I http://your_domain.com

# 检查DNS解析
nslookup your_domain.com

# 检查端口开放
sudo netstat -tlnp | grep :80
sudo netstat -tlnp | grep :443

# 检查Nginx配置语法
sudo nginx -t

# 重新加载Nginx配置
sudo nginx -s reload
```

---

*本指南提供了完整的部署流程，确保网站能够稳定运行在生产环境中。*
