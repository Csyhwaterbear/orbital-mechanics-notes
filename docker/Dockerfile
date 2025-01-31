# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.

# Ubuntu 20.04 (focal)
# https://hub.docker.com/_/ubuntu/?tab=tags&name=focal
ARG ROOT_CONTAINER=ubuntu:focal

FROM $ROOT_CONTAINER

LABEL maintainer="Bryan Weber <bryan.w.weber@gmail.com>"
ARG NB_USER="jovyan"
ARG NB_UID="1000"
ARG NB_GID="100"

# Fix DL4006
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root

# ---- Miniforge installer ----
# Default values can be overridden at build time
# (ARGS are in lower case to distinguish them from ENV)
# Check https://github.com/conda-forge/miniforge/releases
# Conda version
ARG conda_version="4.10.3"
# Miniforge installer patch version
ARG miniforge_patch_number="5"
# Package Manager and Python implementation to use (https://github.com/conda-forge/miniforge)
# - conda only: either Miniforge3 to use Python or Miniforge-pypy3 to use PyPy
# - conda + mamba: either Mambaforge to use Python or Mambaforge-pypy3 to use PyPy
ARG miniforge_python="Mambaforge"
# Miniforge archive to install
ARG miniforge_version="${conda_version}-${miniforge_patch_number}"

# Julia version to install
ARG julia_version="1.6.3"


# Install all OS dependencies for notebook server that starts but lacks all
# features (e.g., download as all possible file formats)
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends \
    tini \
    wget \
    ca-certificates \
    sudo \
    locales \
    fonts-liberation \
    run-one && \
    apt-get clean && rm -rf /var/lib/apt/lists/* && \
    echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

# Configure environment
ENV CONDA_DIR=/opt/conda \
    JULIA_DIR=/opt/julia \
    SHELL=/bin/bash \
    NB_USER="${NB_USER}" \
    NB_UID=${NB_UID} \
    NB_GID=${NB_GID} \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8
ENV PATH="${JULIA_DIR}/bin:${CONDA_DIR}/bin:${PATH}" \
    HOME="/home/${NB_USER}" \
    CONDA_VERSION="${conda_version}" \
    JULIA_VERSION="${julia_version}" \
    MINIFORGE_VERSION="${miniforge_version}"

# Copy a script that we will use to correct permissions after running certain commands
COPY fix-permissions /usr/local/bin/fix-permissions
RUN chmod a+rx /usr/local/bin/fix-permissions

# Enable prompt color in the skeleton .bashrc before creating the default NB_USER
# hadolint ignore=SC2016
RUN sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc && \
    # Add call to conda init script see https://stackoverflow.com/a/58081608/4413446
    echo 'eval "$(command conda shell.bash hook 2> /dev/null)"' >> /etc/skel/.bashrc

# Create NB_USER with name jovyan user with UID=1000 and in the 'users' group
# and make sure these dirs are writable by the `users` group.
RUN echo "auth requisite pam_deny.so" >> /etc/pam.d/su && \
    sed -i.bak -e 's/^%admin/#%admin/' /etc/sudoers && \
    sed -i.bak -e 's/^%sudo/#%sudo/' /etc/sudoers && \
    useradd -l -m -s /bin/bash -N -u "${NB_UID}" "${NB_USER}" && \
    mkdir -p "${CONDA_DIR}" && \
    mkdir -p "${JULIA_DIR}" && \
    chown "${NB_USER}:${NB_GID}" "${CONDA_DIR}" && \
    chown "${NB_USER}:${NB_GID}" "${JULIA_DIR}" && \
    chmod g+w /etc/passwd && \
    fix-permissions "${HOME}" && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "${JULIA_DIR}"

USER ${NB_UID}
ARG PYTHON_VERSION=default

# Setup work directory for backward-compatibility
RUN mkdir "/home/${NB_USER}/work" && \
    fix-permissions "/home/${NB_USER}"

# Install conda as jovyan and check the sha256 sum provided on the download site
WORKDIR /tmp

# Prerequisites installation: conda, mamba, pip, tini
RUN set -x && \
    # Miniforge installer
    miniforge_arch=$(uname -m) && \
    export miniforge_arch && \
    if [ "$miniforge_arch" == "aarm64" ]; then \
      miniforge_arch="arm64"; \
    fi; \
    miniforge_installer="${miniforge_python}-${miniforge_version}-Linux-${miniforge_arch}.sh" && \
    export miniforge_installer && \
    wget --quiet "https://github.com/conda-forge/miniforge/releases/download/${miniforge_version}/${miniforge_installer}" && \
    /bin/bash "${miniforge_installer}" -f -b -p "${CONDA_DIR}" && \
    rm "${miniforge_installer}" && \
    # Conda configuration see https://conda.io/projects/conda/en/latest/configuration.html
    echo "conda ${CONDA_VERSION}" >> "${CONDA_DIR}/conda-meta/pinned" && \
    conda config --system --set auto_update_conda false && \
    conda config --system --set show_channel_urls true && \
    if [[ "${PYTHON_VERSION}" != "default" ]]; then conda install --yes python="${PYTHON_VERSION}"; fi && \
    conda list python | grep '^python ' | tr -s ' ' | cut -d ' ' -f 1,2 >> "${CONDA_DIR}/conda-meta/pinned" && \
    conda install --quiet --yes \
    "conda=${CONDA_VERSION}" \
    'pip' && \
    conda update --all --quiet --yes && \
    conda clean --all -f -y && \
    rm -rf "/home/${NB_USER}/.cache/yarn" && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

# Install Julia official latest version
RUN set -x && \
    julia_arch=$(uname -m) && \
    export julia_arch && \
    if [ "$julia_arch" == "aarm64" ]; then \
      julia_arch="aarch64"; \
    fi; \
    julia_installer="julia-${julia_version}-linux-${julia_arch}.tar.gz" && \
    export julia_installer && \
    wget --quiet "https://julialang-s3.julialang.org/bin/linux/${julia_arch}/${JULIA_VERSION:0:3}/${julia_installer}" && \
    tar -xf "${julia_installer}" && \
    rm "${julia_installer}" && \
    mv julia-${julia_version}/* "${JULIA_DIR}/" && \
    fix-permissions "${JULIA_DIR}"

COPY startup.jl /tmp/
COPY packages.jl /tmp/

RUN julia "/tmp/packages.jl" && \
    fix-permissions "/home/${NB_USER}/.julia"

# Install Jupyter Notebook, Lab, and Hub
# Generate a notebook server config
# Cleanup temporary files
# Correct permissions
# Do all this in a single RUN command to avoid duplicating all of the
# files across image layers when the permissions change
RUN mamba install --quiet --yes \
    'jupyterlab' \
    'matplotlib' \
    'scipy' \
    'numpy' \
    'ipympl' \
    'poliastro' \
    'skyfield' \
    'plotly' && \
    conda clean --all -f -y && \
    jupyter lab --generate-config && \
    jupyter lab clean && \
    rm -rf "/home/${NB_USER}/.cache/yarn" && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

EXPOSE 10000

# Configure container startup
ENTRYPOINT ["tini", "-g", "--"]
CMD ["start-notebook.sh", "--NotebookApp.password=''", "--NotebookApp.token=''"]

# Copy local files as late as possible to avoid cache busting
COPY start.sh start-notebook.sh /usr/local/bin/
# Currently need to have both jupyter_notebook_config and jupyter_server_config to support classic and lab
COPY jupyter_server_config.py /etc/jupyter/

# Fix permissions on /etc/jupyter as root
USER root

RUN chmod a+rx /usr/local/bin/start.sh
RUN chmod a+rx /usr/local/bin/start-notebook.sh
# Prepare upgrade to JupyterLab V3.0 #1205
RUN fix-permissions /etc/jupyter/

# Switch back to jovyan to avoid accidental container runs as root
USER ${NB_UID}

# The startup.jl file sets the LD_PRELOAD environment variable to avoid problems with
# TLS memory access when using Luxor (I think it's Luxor anyways)
# Setup work directory for backward-compatibility
RUN mkdir -p "/home/${NB_USER}/.julia/config" && \
    cp "/tmp/startup.jl" "${HOME}/.julia/config" && \
    fix-permissions "/home/${NB_USER}/.julia"

WORKDIR "${HOME}"
