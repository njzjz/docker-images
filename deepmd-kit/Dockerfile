ARG BASE_IMAGE=ubuntu:focal
FROM $BASE_IMAGE

ARG DOCKERTAG
ARG VERSION
ARG INSTALLER_VERSION
ARG TINI_VERSION=v0.18.0

ENV DEEPMD_DIR=/opt/deepmd-kit
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8
ENV PATH=${DEEPMD_DIR}/bin:${PATH}

# Install just enough for conda to work
RUN apt-get update > /dev/null && \
    apt-get install --no-install-recommends --yes \
        wget bzip2 ca-certificates \
        git > /dev/null && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Keep $HOME clean (no .wget-hsts file), since HSTS isn't useful in this context
RUN wget --no-hsts --quiet https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini -O /usr/local/bin/tini && \
    chmod +x /usr/local/bin/tini

# 1. Install deepmd-kit from GitHub releases
# 2. Apply some cleanup tips from https://jcrist.github.io/conda-docker-tips.html
#    Particularly, we remove pyc and a files. The default install has no js, we can skip that
RUN wget --no-hsts --quiet https://github.com/deepmd-kit-recipes/installer/releases/download/v${VERSION}${INSTALLER_VERSION}/deepmd-kit-${VERSION}-${VARIANT}-Linux-x86_64.sh -O /tmp/installer.sh && \
    /bin/bash /tmp/installer.sh -b -p ${DEEPMD_DIR} && \
    rm /tmp/installer.sh && \
    conda clean -tipsy && \
    find ${DEEPMD_DIR} -follow -type f -name '*.a' -delete && \
    find ${DEEPMD_DIR} -follow -type f -name '*.pyc' -delete && \
    conda clean -afy

# Activate base by default when running as any *non-root* user as well
# Good security practice requires running most workloads as non-root
# This makes sure any non-root users created also have base activated
# for their interactive shells.
RUN echo ". ${DEEPMD_DIR}/etc/profile.d/conda.sh && conda activate base" >> /etc/skel/.bashrc

# Activate base by default when running as root as well
# The root user is already created, so won't pick up changes to /etc/skel
RUN echo ". ${DEEPMD_DIR}/etc/profile.d/conda.sh && conda activate base" >> ~/.bashrc

ENTRYPOINT ["tini", "--"]
CMD [ "/bin/bash" ]
