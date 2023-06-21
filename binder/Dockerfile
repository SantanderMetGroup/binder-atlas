FROM santandermetgroup/binder-atlas:climate4r-v2.5.4
# Avoid prompts from apt
ENV DEBIAN_FRONTEND=noninteractive

ENV LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8

# Use bash as default shell, rather than sh
ENV SHELL=/bin/bash

# Set up user
ARG NB_USER
ARG NB_UID
ENV USER=${NB_USER} \
    HOME=/home/${NB_USER}

RUN groupadd \
        --gid ${NB_UID} \
        ${NB_USER} && \
    useradd \
        --comment "Default user" \
        --create-home \
        --gid ${NB_UID} \
        --no-log-init \
        --shell /bin/bash \
        --uid ${NB_UID} \
        ${NB_USER}

EXPOSE 8888

# Environment variables required for build
ENV APP_BASE=/srv
ENV CONDA_DIR=${APP_BASE}/conda
ENV NB_PYTHON_PREFIX=${CONDA_DIR}/envs/notebook
ENV NPM_DIR=${APP_BASE}/npm
ENV NPM_CONFIG_GLOBALCONFIG=${NPM_DIR}/npmrc
ENV NB_ENVIRONMENT_FILE=/tmp/env/environment.lock
ENV MAMBA_ROOT_PREFIX=${CONDA_DIR}
ENV MAMBA_EXE=${CONDA_DIR}/bin/mamba
ENV CONDA_PLATFORM=linux-64
ENV KERNEL_PYTHON_PREFIX=${NB_PYTHON_PREFIX}
# Special case PATH
ENV PATH=${NB_PYTHON_PREFIX}/bin:${CONDA_DIR}/bin:${NPM_DIR}/bin:${PATH}
# If scripts required during build are present, copy them

RUN mkdir -p ${NPM_DIR} && \
chown -R ${NB_USER}:${NB_USER} ${NPM_DIR}


# ensure root user after build scripts
USER root

# Allow target path repo is cloned to be configurable
ARG REPO_DIR=${HOME}
ENV REPO_DIR=${REPO_DIR}
# Create a folder and grant the user permissions if it doesn't exist
RUN if [ ! -d "${REPO_DIR}" ]; then \
        /usr/bin/install -o ${NB_USER} -g ${NB_USER} -d "${REPO_DIR}"; \
    fi

WORKDIR ${REPO_DIR}
RUN chown ${NB_USER}:${NB_USER} ${REPO_DIR}

# We want to allow two things:
#   1. If there's a .local/bin directory in the repo, things there
#      should automatically be in path
#   2. postBuild and users should be able to install things into ~/.local/bin
#      and have them be automatically in path
#
# The XDG standard suggests ~/.local/bin as the path for local user-specific
# installs. See https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html
ENV PATH=${HOME}/.local/bin:${REPO_DIR}/.local/bin:${PATH}

# The rest of the environment
ENV CONDA_DEFAULT_ENV=${KERNEL_PYTHON_PREFIX}
# Run pre-assemble scripts! These are instructions that depend on the content
# of the repository but don't access any files in the repository. By executing
# them before copying the repository itself we can cache these steps. For
# example installing APT packages.
# If scripts required during build are present, copy them

USER ${NB_USER}
RUN pip install --no-cache \
    jupyter-rsession-proxy==2.0.1 \
    jupyter-shiny-proxy==1.1

USER root
RUN install -o ${NB_USER} -g ${NB_USER} -d /var/log/shiny-server && \
install -o ${NB_USER} -g ${NB_USER} -d /var/lib/shiny-server && \
install -o ${NB_USER} -g ${NB_USER} /dev/null /var/log/shiny-server.log && \
install -o ${NB_USER} -g ${NB_USER} /dev/null /var/run/shiny-server.pid

RUN echo auth-none=1 >> /etc/rstudio/rserver.conf && \
echo auth-minimum-user-id=0 >> /etc/rstudio/rserver.conf && \
echo "rsession-which-r=${NB_PYTHON_PREFIX}/bin/R" >> /etc/rstudio/rserver.conf && \
echo "rsession-ld-library-path=${NB_PYTHON_PREFIX}/lib" >> /etc/rstudio/rserver.conf && \
echo www-frame-origin=same >> /etc/rstudio/rserver.conf

USER ${NB_USER}
RUN R --quiet -e "IRkernel::installspec(prefix='${NB_PYTHON_PREFIX}')"


# ensure root user after preassemble scripts
USER root

# We always want containers to run as non-root
USER ${NB_USER}

# Add start script
# Add entrypoint
ENV PYTHONUNBUFFERED=1
ENTRYPOINT ["/usr/local/bin/repo2docker-entrypoint"]
# Specify the default command to run
CMD ["jupyter", "notebook", "--ip", "0.0.0.0"]
