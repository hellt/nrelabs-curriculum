FROM ubuntu:18.04

RUN apt-get update && apt-get install -y openssh-server python python-pip dnsutils iputils-ping git vim curl util-linux sshpass nano jq libxml2-utils
RUN mkdir /var/run/sshd

# Antidote user
RUN mkdir -p /home/antidote
RUN useradd antidote -p antidotepassword
RUN chown antidote:antidote /home/antidote
RUN chsh antidote --shell=/bin/bash
RUN echo 'antidote:antidotepassword' | chpasswd
RUN echo 'root:$(uuidgen)' | chpasswd

# Adjust MOTD
RUN rm -f /etc/update-motd.d/*
RUN rm -f /etc/legal
ADD motd.sh /etc/update-motd.d/00-antidote-motd

# Disable root Login
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config
RUN sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config

# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

# Disable su for everyone not in the wheel group (no one is in the wheel group)
RUN echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su

# https://stackoverflow.com/questions/36292317/why-set-visible-now-in-etc-profile
ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile

ADD requirements.txt /requirements.txt
RUN pip install -r /requirements.txt

# The snapshots directory is apparently not being set up during installation so we'll do it here
RUN mkdir -p /home/antidote/jsnapy/snapshots && chown -R antidote:antidote /home/antidote/jsnapy

# Temporary fix to get NAPALM working. Long-term we should convert utility to use python 3
RUN pip install --upgrade netmiko==2.4.2

RUN curl -L -o yq https://github.com/mikefarah/yq/releases/download/3.3.2/yq_linux_amd64 && chmod +x yq && mv yq /usr/local/bin
RUN wget https://github.com/sharkdp/bat/releases/download/v0.15.4/bat_0.15.4_amd64.deb && dpkg -i bat_0.15.4_amd64.deb

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
