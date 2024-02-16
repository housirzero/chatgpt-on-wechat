# 如何在阿里云主机上部署
## 阿里云环境信息
镜像ID: centos_7_9_x64_20G_alibase_20240125.vhd

## 编译安装openssl-1.1.1w
```
wget https://www.openssl.org/source/old/1.1.1/openssl-1.1.1w.tar.gz
tar zxf openssl-1.1.1w.tar.gz
cd openssl-1.1.1w
./config --prefix=/usr/local --openssldir=/usr/local/openssl-1.1.1w
make && make install

# 备份openssl-1.0.2k-fips
mv /usr/bin/openssl /usr/bin/openssl-1.0.2k-fips

# 创建openssl软连接
ln -s /usr/local/bin/openssl /usr/bin/openssl
ln -s /usr/local/include/openssl/ /usr/include/openssl
# 修改动态链接库路径
echo "/usr/local/lib64" >> /etc/ld.so.conf.d/openssl.conf
# 生效动态链接库
ldconfig -v
# 检查版本
openssl version
```


## 配置ssh免密码登录
```
ssh-keygen -t rsa -C "your_email"
cat /root/.ssh/id_rsa.pub
```

## 安装python3
**实测安装3.11.8可以的成功，安装3.8.3和3.12.2都有问题**
```
# 下载
wget https://www.python.org/ftp/python/3.11.8/Python-3.11.8.tgz

# 设置python版本，用法见install_py
function change_py_pip()
{
    INSTALL_DIR=$1
    rm -f /usr/bin/python
    ln -s ${INSTALL_DIR}/bin/python3 /usr/bin/python
    python -V

    rm -f /usr/bin/pip
    ln -s ${INSTALL_DIR}/bin/pip3 /usr/bin/pip
    pip install --upgrade pip
    pip -V
}

function install_py()
{
    version=$1
    tar -zxvf Python-${version}.tgz
    mkdir /usr/local/python${version}
    cd Python-${version}
    ./configure --prefix=/usr/local/python${version}
    make && make install

    change_py_pip /usr/local/python${version}
}
```

## yum无法使用python3，需要修改yum的配置文件
```
sed -i "s/\/usr\/bin\/python/\/usr\/bin\/python2/g" /usr/bin/yum
sed -i "s/\/usr\/bin\/python/\/usr\/bin\/python2/g" /usr/libexec/urlgrabber-ext-down
sed -i "s/\/usr\/bin\/python/\/usr\/bin\/python2/g" /usr/bin/yum-config-manager
```

## 参考
[CentOS安装python3后使用yum报错](https://cloud.tencent.com/developer/article/1800245)
https://www.jianshu.com/p/69681655309b
[centos7 安装Python3.8.3](https://developer.aliyun.com/article/791131)
[OpenSSL升级](https://www.bilibili.com/video/BV1cM411R7r8/?vd_source=87da5d8a68a3ba3764db02d91e61afb8)
[Python3.7.0以上版本安装pip报错ModuleNotFoundError: No module named '_ctypes'解决方法](https://www.jianshu.com/p/69681655309b)