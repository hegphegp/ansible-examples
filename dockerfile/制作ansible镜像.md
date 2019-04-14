```
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

docker build -t ansible:2.6.12 .
```