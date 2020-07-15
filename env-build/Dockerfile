FROM fedora:32

RUN mkdir /handson &&\
    dnf install -y wget &&\
    wget https://pastebin.com/raw/MguxpzqA -O /handson/pod-example.yaml &&\ 
    dnf install -y origin-cluster-capacity &&\
    chgrp -R 0 /handson && \
    chmod -R g=u /handson 
