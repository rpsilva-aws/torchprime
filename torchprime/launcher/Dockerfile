# syntax=docker/dockerfile:experimental
# Use torch_xla Python 3.10 nightly as the base image
FROM us-central1-docker.pkg.dev/tpu-pytorch-releases/docker/xla:nightly_3.10_tpuvm_cxx11

# Install system dependencies
RUN apt-get update && apt-get upgrade -y
RUN apt-get update && apt-get install -y curl gnupg

# Add the Google Cloud SDK package repository
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
RUN curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -

# Add the Cloud Storage FUSE distribution URL as a package source
RUN echo "deb https://packages.cloud.google.com/apt gcsfuse-bullseye main" | tee /etc/apt/sources.list.d/gcsfuse.list
RUN curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

# Install the Google Cloud SDK and GCS fuse
RUN apt-get update && apt-get install -y google-cloud-sdk git fuse gcsfuse && gcsfuse -v

# Set the default Python version to 3.10
RUN update-alternatives --install /usr/bin/python3 python3 /usr/local/bin/python3.10 1

WORKDIR /workspaces

# Install torchax
RUN git clone https://github.com/pytorch/xla.git
WORKDIR /workspaces/xla/torchax
RUN pip install torch_xla[pallas] \
    -f https://storage.googleapis.com/jax-releases/jax_nightly_releases.html \
    -f https://storage.googleapis.com/jax-releases/jaxlib_nightly_releases.html
RUN pip install -e .

# Install torchprime
# Optimization: we rerun `pip install -e .` only if `pyproject.toml` changes.
# Copy only the installation-related files first to make Docker cache them separately.
WORKDIR /workspaces/torchprime
COPY pyproject.toml /workspaces/torchprime/
RUN pip install -e .

# Now copy the rest of the repo
COPY . /workspaces/torchprime
# This should not install any packages. Only symlink the source code.
RUN pip install --no-deps -e .

ENV LIBTPU_INIT_ARGS "--xla_tpu_scoped_vmem_limit_kib=98304 --xla_enable_async_all_gather=true --xla_tpu_overlap_compute_collective_tc=true --xla_tpu_enable_async_collective_fusion_multiple_steps=true --xla_tpu_enable_async_collective_fusion=true --xla_tpu_enable_async_collective_fusion_fuse_all_gather=true"
