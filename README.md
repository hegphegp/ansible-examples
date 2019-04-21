
```
##### 制作Dockerfile文件
tee Dockerfile <<-'EOF'
FROM alpine:3.8

RUN echo "===> Adding Python runtime..."  && \
    echo -e "http://mirrors.aliyun.com/alpine/v3.8/main\nhttp://mirrors.aliyun.com/alpine/v3.8/community" > /etc/apk/repositories && \
    mkdir -p ~/.pip && echo -e "[global]\nindex-url=http://mirrors.aliyun.com/pypi/simple\n[install]\ntrusted-host=mirrors.aliyun.com">> ~/.pip/pip.conf && \
    apk --update add python py-pip openssl ca-certificates sshpass   && \
    apk --update add --virtual build-dependencies python-dev libffi-dev openssl-dev build-base          && \
    pip install --upgrade pip cffi                            && \
    \
    echo "===> Installing Ansible..."  && \
    pip install ansible==2.6.12        && \
    \
    echo "===> Installing handy tools..."          && \
    pip install --upgrade pycrypto                 && \
    apk --update add bash openssh-client rsync     && \
    \
    echo "===> Removing package list..."  && \
    apk del build-dependencies            && \
    rm -rf /var/cache/apk/*               && \
    rm -rf /root/.cache

CMD [ "ansible", "--version" ]
EOF
##### 制作Dockerfile文件结束

docker build -t ansible:2.6.12 .


##### 制作Dockerfile文件
tee Dockerfile <<-'EOF'
FROM centos:7.6.1810

ENV ROOT_PASSWORD root

RUN curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo ; \
    echo "export LC_ALL=en_US.UTF-8" >> /etc/profile ; \
    yum install -y openssh-server openssh-clients firewalld ; \
    ssh-keygen -A ; \
    echo "root:${ROOT_PASSWORD}" | chpasswd ; \
    sed -i 's/UsePAM yes/UsePAM no/g' /etc/ssh/sshd_config; \
    sed -i 's/#UseDNS yes/UseDNS no/g' /etc/ssh/sshd_config; \
    sed -i 's/#PermitRootLogin yes/PermitRootLogin yes/' /etc/ssh/sshd_config; \
    yum clean all

EXPOSE 22

CMD ["/usr/sbin/init"]
EOF
##### 制作Dockerfile文件结束

docker build -t centos-sshd:7.6.1810 .


docker stop web1 web2 web3 ansible
docker rm web1 web2 web3 ansible
docker network rm ansible-network
docker network create --subnet=10.10.58.0/24 ansible-network
docker run --privileged -itd --restart always --net ansible-network --ip 10.10.58.100 --name ansible -v $PWD:/ansible ansible:2.6.12 sh
docker run --privileged -itd --restart always --net ansible-network --ip 10.10.58.101 --name web1 centos-sshd:7.6.1810 /usr/sbin/init
docker run --privileged -itd --restart always --net ansible-network --ip 10.10.58.102 --name web2 centos-sshd:7.6.1810 /usr/sbin/init
docker run --privileged -itd --restart always --net ansible-network --ip 10.10.58.103 --name web3 centos-sshd:7.6.1810 /usr/sbin/init
docker exec -it ansible sh
cd /ansible
ansible-playbook mian.yml
# ansible-playbook main.yml --syntax-check    # 检查yaml文件的语法是否正确
# ansible-playbook main.yml --list-task       # 检查tasks任务
# ansible-playbook main.yml --list-hosts      # 检查生效的主机
# ansible-playbook main.yml --start-at-task='Copy Nginx.conf'     # 指定从某个task开始运行
```