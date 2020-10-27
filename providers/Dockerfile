ARG BASECONTAINER
ARG BASECONTAINER_REPO=norseto/${BASECONTAINER:-"dev-base:k8s"}
FROM ${BASECONTAINER_REPO}

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

ARG INST_AZURECLI
ARG INST_AWSCLI
ARG INST_GCPSDK

# Configure apt and install packages
RUN apt-get update \
    #--------------------------------------
    # Google Cloud SDK
    #--------------------------------------
    && if [ "x${INST_GCPSDK}" != "x" ] ; then \
      true \
      && apt-get install -y --no-install-recommends google-cloud-sdk \
      && gcloud --version \
    ; fi \
    #
    #--------------------------------------
    # AWS CLI
    #--------------------------------------
    && if [ "x${INST_AWSCLI}" != "x" ] ; then \
      true \
      && curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" \
      && unzip awscliv2.zip && rm -rf awscliv2.zip && ./aws/install \
      && rm -rf ./aws \
      && curl -o aws-iam-authenticator https://amazon-eks.s3.us-west-2.amazonaws.com/1.17.9/2020-08-04/bin/linux/amd64/aws-iam-authenticator \
      && install ./aws-iam-authenticator /usr/local/bin && rm -f ./aws-iam-authenticator \
      && curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp \
      && install /tmp/eksctl /usr/local/bin \
      && aws --version \
      && echo "=== eksctl version ===" && eksctl version \
    ; fi \
    #
    #--------------------------------------
    # Azure CLI
    #--------------------------------------
    && if [ "x${INST_AZURECLI}" != "x" ] ; then \
      true \
      && apt-get install --no-install-recommends -y azure-cli \
      && az --version \
    ; fi \
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=
