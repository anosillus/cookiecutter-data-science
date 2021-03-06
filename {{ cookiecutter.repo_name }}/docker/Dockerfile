FROM debian:stable-20200908-slim

LABEL author="anosillus"
USER root

RUN apt-get update --fix-missing && apt-get -yq dist-upgrade && \
    apt-get install -y \
      bzip2 \
      ca-certificates \
      fonts-liberation \
      git \
      libglib2.0-0\
      libsm6 \
      libxext6 \
      libxrender1 \
      locales \
      mercurial \
      subversion \
      sudo \
      wget \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

ENV CONDA_DIR=/opt/conda \
    SHELL=/bin/bash \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8

ENV PATH=$CONDA_DIR/bin:$PATH
ENV HOME=/home/
WORKDIR $HOME

RUN wget --quiet https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /tmp/miniconda.sh && \
    /bin/bash /tmp/miniconda.sh -f -b -p $CONDA_DIR && \
    rm /tmp/miniconda.sh && \
    ln -s $CONDA_DIR/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    echo ". /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc && \
    $CONDA_DIR/bin/conda config --system --prepend channels conda-forge && \
    $CONDA_DIR/bin/conda update --all --quiet --yes && \
    conda clean -tipsy && \
    # conda build purge-all && \
    rm -rf /home/.cache/yarn

RUN apt-get update --fix-missing && apt-get -yq dist-upgrade && \
    apt-get install -y \
      curl \
      grep \
      sed \
      dpkg \
      libxml2 \
      libxml2-dev \
      libxslt-dev \
      python-lxml \
    && TINI_VERSION=`curl https://github.com/krallin/tini/releases/latest | grep -o "/v.*\"" | sed 's:^..\(.*\).$:\1:'` && \
    curl -L "https://github.com/krallin/tini/releases/download/v${TINI_VERSION}/tini_${TINI_VERSION}.deb" > tini.deb && \
    dpkg -i tini.deb && \
    rm tini.deb && \
    rm -rf /var/lib/apt/lists/* && \
    apt-get clean
# "

RUN mkdir -p /home/.jupyter && { \
    echo "c.NotebookApp.allow_root = True"; \
    echo "c.NotebookApp.ip = '0.0.0.0'"; \
    echo "c.NotebookApp.token = ''"; \
    echo "c.JupyterLabTemplates.template_dirs = ['/home/templates/']"; \
    echo "c.JupyterLabTemplates.include_default = True"; \
    echo "c.LatexConfig.latex_command = 'xetex'"; \
    } | tee /home/.jupyter/jupyter_notebook_config.py

RUN conda install -c conda-forge jupyterlab jupyterlab-git jupyter-lsp nodejs python-language-server ipympl jupyterhub nbdime jupyterlab_templates ipywidgets && \
    # conda install -c anaconda libxml2 ipykernel && \
    jupyter labextension install nbdime-jupyterlab --no-build && \
    jupyter labextension install @jupyter-widgets/jupyterlab-manager --no-build && \
    jupyter labextension install @jupyterlab/latex --no-build && \
    jupyter labextension install @krassowski/jupyterlab-lsp --no-build && \
    jupyter labextension install @jupyterlab/toc --no-build && \
    jupyter labextension install jupyter-matplotlib --no-build && \
    jupyter labextension install @jupyterlab/hub-extension --no-build && \
    jupyter labextension install jupyterlab_templates --no-build && \
    jupyter labextension install @jupyterlab/git --no-build && \
    jupyter lab build && \
    jupyter serverextension enable --py jupyterlab_templates && \
    jupyter serverextension enable --py jupyterlab_git && \
    conda clean -tipsy && \
    jupyter lab clean && \
    jlpm cache clean && \
    npm cache clean --force && \
    rm -rf /home/.cache/yarn && \
    rm -rf $CONDA_DIR/share/jupyter/lab/staging && \
    rm -rf /home/.node-gyp && \
    rm -rf /home/.local

COPY ./environment_base.yaml /tmp/environment_base.yaml
RUN conda env create -f /tmp/environment_base.yaml && \
    conda clean --all -f -y && \
    npm cache clean --force && \
    rm -rf /home/.cache/yarn && \
    echo "conda activate $(head -1 /tmp/environment_base.yaml | cut -d' ' -f2 | tr -d '\r')" >> /home/.bashrc

ENV PATH /opt/conda/envs/$(head -1 /tmp/environment_base.yaml | cut -d' ' -f2 | tr -d '\r')/bin:$PATH

RUN apt-get update --fix-missing && \
    apt-get install -y fish texlive-xetex fonts-ipafont fonts-ipaexfont fonts-noto-cjk && \
    rm -rf /var/lib/apt/lists/* && \
    apt-get clean && \
    mkdir -p /home/.config/fish && \
    touch /home/.config/fish/fish.config && \
    echo "source /opt/conda/etc/fish/conf.d/conda.fish" >> /home/.config/fish/fish.config && \
    echo "conda activate $(head -1 /tmp/environment_base.yaml | cut -d' ' -f2 | tr -d '\r')" >> /home/.config/fish/fish.config

SHELL ["/bin/bash", "-c"]
ENV BASH_ENV /home/.bashrc
RUN conda install -n $(head -1 /tmp/environment_base.yaml | cut -d' ' -f2 | tr -d '\r') notebook ipykernel && \
  /opt/conda/envs/$(head -1 /tmp/environment_base.yaml | cut -d' ' -f2 | tr -d '\r')/bin/python -m ipykernel install --name  $(head -1 /tmp/environment_base.yaml | cut -d' ' -f2 | tr -d '\r') --display-name  $(head -1 /tmp/environment_base.yaml | cut -d' ' -f2 | tr -d '\r') && \
  conda clean --all -f -y && \
  npm cache clean --force && \
  rm -rf ~/.cache/yarn

RUN mkdir /home/playground && \
    mkdir /home/data && \
    mkdir /home/src && \
    mkdir /home/models && \
    mkdir /home/reports && \
    mkdir /home/references && \
    mkdir /home/templates

VOLUME /home/playground
VOLUME /home/templates
VOLUME /home/data
VOLUME /home/src
VOLUME /home/models
VOLUME /home/reports
VOLUME /home/references

EXPOSE 8888
ENTRYPOINT [ "/usr/bin/tini", "--" ]
