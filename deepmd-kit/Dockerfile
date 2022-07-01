ARG BASE_IMAGE=ubuntu:focal
FROM $BASE_IMAGE

LABEL maintainer="Jinzhe Zeng <jinzhe.zeng@rutgers.edu>" \
      org.opencontainers.image.authors="Jinzhe Zeng <jinzhe.zeng@rutgers.edu>" \
      org.opencontainers.image.url="https://github.com/deepmodeling/deepmd-kit" \
      org.opencontainers.image.documentation="https://github.com/deepmodeling/deepmd-kit" \
      org.opencontainers.image.vendor="deepmodeling" \
      org.opencontainers.image.description="You can use `dp -h` or `lmp -h` to see the help." \
      org.opencontainers.image.source="https://github.com/deepmodeling/deepmd-kit"

ARG VARIANT
ARG VERSION
ARG TINI_VERSION=v0.19.0

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
# remove pkgs and include and ensure it works
RUN if [ "${VARIANT}" != "cuda11.6_gpu" ] ; then \
      wget --no-hsts --quiet https://github.com/deepmodeling/deepmd-kit/releases/download/v${VERSION}/deepmd-kit-${VERSION}-${VARIANT}-Linux-x86_64.sh -O /tmp/installer.sh ; else \
      wget --no-hsts --quiet https://github.com/deepmodeling/deepmd-kit/releases/download/v${VERSION}/deepmd-kit-${VERSION}-${VARIANT}-Linux-x86_64.sh.0 -O /tmp/installer.sh.0 && \
      wget --no-hsts --quiet https://github.com/deepmodeling/deepmd-kit/releases/download/v${VERSION}/deepmd-kit-${VERSION}-${VARIANT}-Linux-x86_64.sh.1 -O /tmp/installer.sh.1 && \
	  cat /tmp/installer.sh.0 /tmp/installer.sh.1 > /tmp/installer.sh ; fi && \
    CONDA_OVERRIDE_CUDA=11.6 /bin/bash /tmp/installer.sh -b -p ${DEEPMD_DIR} && \
    rm /tmp/installer.sh* && \
    rm -rf ${DEEPMD_DIR}/pkgs && \
    rm -rf ${DEEPMD_DIR}/share && \
    rm -rf ${DEEPMD_DIR}/include && \
    ${DEEPMD_DIR}/bin/dp --version && \
    ${DEEPMD_DIR}/bin/lmp -h && \
    find ${DEEPMD_DIR} -follow -type f -name '*.a' -delete && \
    find ${DEEPMD_DIR} -follow -type f -name '*.pyc' -delete

# Activate base by default when running as any *non-root* user as well
# Good security practice requires running most workloads as non-root
# This makes sure any non-root users created also have base activated
# for their interactive shells.
RUN echo ". ${DEEPMD_DIR}/etc/profile.d/conda.sh && conda activate" >> /etc/skel/.bashrc

# Activate base by default when running as root as well
# The root user is already created, so won't pick up changes to /etc/skel
RUN echo ". ${DEEPMD_DIR}/etc/profile.d/conda.sh && conda activate" >> ~/.bashrc

ENTRYPOINT ["tini", "--"]
CMD [ "/bin/bash" ]
