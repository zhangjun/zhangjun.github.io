FROM     centos
LABEL    com.zhangjun.image.authors="ewalker.zj@gmail.com"

ENV TZ "Asia/Shanghai"

RUN yum install -y curl wget tar bzip2 unzip vim-enhanced passwd sudo yum-utils hostname net-tools rsync man && \
   yum install -y gcc gcc-c++ git make automake cmake patch python-devel  && \
   yum install -y --enablerepo=epel pwgen python-pip && \
   yum install -y npm && npm install -g hexo && \ 
   yum clean all