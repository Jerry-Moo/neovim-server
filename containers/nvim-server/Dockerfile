#  _   _                 _             ____
# | \ | | ___  _____   _(_)_ __ ___   / ___|  ___ _ ____   _____ _ __
# |  \| |/ _ \/ _ \ \ / / | '_ ` _ \  \___ \ / _ \ '__\ \ / / _ \ '__|
# | |\  |  __/ (_) \ V /| | | | | | |  ___) |  __/ |   \ V /  __/ |
# |_| \_|\___|\___/ \_/ |_|_| |_| |_| |____/ \___|_|    \_/ \___|_|
#
# https://github.com/yqlbu/neovim-server
#
# Copyright (C) 2020-2021 yqlbu <https://hikariai.net>
#
# This is a open-source software, liscensed under the MIT License.
# See /License for more information.

FROM wuchujie/wetty:latest

ENV TZ=America/LosAngeles \
    PORT=3000 \
    UID=998 \
    GID=998 \
    USER=guest \
    ADDUSER=true \
    SECRET=password \
    SHELL=/bin/bash \
    SUDO=false \
    SSL=false \
    CONTAINER=neovim-server

RUN ARCH=$(dpkg --print-architecture) && echo $ARCH > ./ARCH

# RUN sudo add-apt-repository ppa:neovim-ppa/unstable -y
# RUN sudo add-apt-repository ppa:lazygit-team/release -y
RUN sudo apt-get update && sudo apt-get install -y --no-install-recommends \
    ninja-build gettext libtool libtool-bin autoconf automake cmake g++ pkg-config unzip curl doxygen lua5.3 liblua5.3-dev wget python3-dev python3-pip golang ruby ruby-dev ruby-bundler tmux zsh ranger
# Install Neovim from source.
RUN mkdir -p /root/TMP
RUN cd /root/TMP && git clone https://github.com/neovim/neovim
RUN cd /root/TMP/neovim && make CMAKE_BUILD_TYPE=RelWithDebInfo && sudo make install
RUN rm -rf /root/TMP
# Install lazygit
RUN MY_FLAVOR=Linux_x86_64; curl -s -L $(curl -s https://api.github.com/repos/jesseduffield/lazygit/releases/latest | grep browser_download_url | cut -d '"' -f 4 | grep -i "$MY_FLAVOR") | sudo tar xzf - -C /usr/local/bin lazygit
RUN rm -rf /usr/bin/pip && ln -s /usr/bin/pip3 /usr/bin/pip && \
    rm -rf /usr/bin/python && ln -s /usr/bin/python3 /usr/bin/python
RUN pip install -U pip && pip install -U \
    pynvim neovim-remote pylint
RUN gem install neovim # Install neovim ruby support
RUN if [ "$(cat ./ARCH)" = "amd64" ] ; then \
      wget https://github.com/wfxr/code-minimap/releases/download/v0.6.0/code-minimap_0.6.0_amd64.deb; \
      dpkg -i code-minimap_0.6.0_amd64.deb; \
    else \
      wget https://github.com/wfxr/code-minimap/releases/download/v0.6.0/code-minimap_0.6.0_arm64.deb; \
      dpkg -i code-minimap_0.6.0_arm64.deb; \
    rm -rf code-minimap_0.6.0*.deb; \
    fi
RUN npm install -g neovim vim-node-rpc
RUN npm install -g instant-markdown-d@next
RUN npm install -g typescript
RUN npm install -g bash-language-server
RUN sudo apt-get clean && \
    rm -rf /var/lib/apt/list/* /tmp/* /var/tmp/* && \
    rm -rf ./ARCH

WORKDIR /usr/src/app

EXPOSE 3000

ADD ./bootstrap.sh /usr/local/sbin
ADD ./zsh-in-docker.sh /usr/local/sbin
RUN chmod u+x /usr/local/sbin/bootstrap.sh
RUN chmod u+x /usr/local/sbin/zsh-in-docker.sh
COPY ./nvim/. /usr/src/app/nvim
COPY ./plugins/. /usr/src/app/plugins

ENTRYPOINT [ "bootstrap.sh" ]
CMD [ "wetty" ]
