ARG BASECONTAINER
ARG BASECONTAINER_REPO=norseto/${BASECONTAINER:-"dev-base:k8s"}

FROM golang:latest as docs
RUN GO111MODULE="on" go get github.com/terraform-docs/terraform-docs@v0.10.1
RUN go get -u github.com/tfsec/tfsec/cmd/tfsec

FROM ${BASECONTAINER_REPO}

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Terraform and tflint versions
ARG TERRAFORM_VERSION=0.13.5
ARG TFLINT_VERSION=0.20.2
ARG TERRAGRUNT_VERSION=0.25.2

ARG INST_TERRAFORM=1

COPY --from=docs /go/bin/terraform-docs /usr/local/bin/
COPY --from=docs /go/bin/tfsec /usr/local/bin/
# Configure apt and install packages
RUN apt-get update \
    #
    #--------------------------------------
    # Install Terraform, tflint
    #--------------------------------------
    && if [ "x${INST_TERRAFORM}" != "x" ] ; then \
      true \
      && curl -sSL -o /tmp/terraform.zip https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip \
      && unzip /tmp/terraform.zip && rm -f /tmp/terraform.zip \
      && install terraform /usr/local/bin && rm -r terraform \
      && curl -sSL -o /tmp/tflint.zip https://github.com/wata727/tflint/releases/download/v${TFLINT_VERSION}/tflint_linux_amd64.zip \
      && unzip /tmp/tflint.zip && rm -f /tmp/tflint.zip \
      && install tflint /usr/local/bin && rm -r tflint \
      # && cd ~ \
      # && apt-get install -y graphviz \
      # Install terragrunt
      && curl -L https://github.com/gruntwork-io/terragrunt/releases/download/v${TERRAGRUNT_VERSION}/terragrunt_linux_amd64 -O \
      && install ./terragrunt_linux_amd64 /usr/local/bin/terragrunt && rm -f ./terragrunt_linux_amd64 \
      && terraform --version && tflint --version && terragrunt --version \
    ; fi \
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=
