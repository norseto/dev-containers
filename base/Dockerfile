ARG BASECONTAINER
ARG BASECONTAINER_REPO=mcr.microsoft.com/vscode/devcontainers/${BASECONTAINER:-"base:buster"}
FROM ${BASECONTAINER_REPO}

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

ARG HELM_VERSION=3.4.1
ARG ISTIO_VERSION=1.7.5
ARG ARGOCD_VERSION=1.7.10
ARG LINKERD2_VERSION=stable-2.9.2
ARG KN_VERSION=0.20.0

ARG INST_DOCKERCLI=1
ARG INST_K8SCLIENT=1
ARG INST_MINIKUBE=1

# Configure apt and install packages
RUN apt-get update && apt-get install -y --no-install-recommends bash-completion \
    && echo "deb [arch=amd64] https://download.docker.com/linux/$(lsb_release -is | tr '[:upper:]' '[:lower:]') $(lsb_release -cs) stable" | \
                tee -a /etc/apt/sources.list.d/docker.list \
    && curl -fsSL https://download.docker.com/linux/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/gpg | apt-key add - 2>/dev/null \
    && curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key --keyring /usr/share/keyrings/cloud.google.gpg add - \
    && curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - \
    && echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | \
                tee -a /etc/apt/sources.list.d/google-cloud-sdk.list \
    && echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" |  tee -a /etc/apt/sources.list.d/kubernetes.list \
    && echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $(lsb_release -cs) main" > /etc/apt/sources.list.d/azure-cli.list \
    && curl -sL https://packages.microsoft.com/keys/microsoft.asc | apt-key add - 2>/dev/null \
    && apt-get update \
    \
    #--------------------------------------
    # Docker CLI
    #--------------------------------------
    && if [ "x${INST_DOCKERCLI}" != "x" ] ; then \
      true \
      && apt-get install -y --no-install-recommends docker-ce-cli \
      && docker --version \
    ; fi \
    \
    #--------------------------------------
    # Kubectl
    #--------------------------------------
    && if [ "x${INST_K8SCLIENT}" != "x" ] ; then \
      true \
      && apt-get install -y --no-install-recommends kubectl vim-tiny \
      && echo 'source /etc/profile.d/bash_completion.sh' >> /root/.bashrc \
      && echo 'source <(kubectl completion bash)' >> /root/.bashrc \
      \
      #--------------------------------------
      # skaffold, kustomize
      #--------------------------------------
      && curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64 \
      && install skaffold /usr/local/bin/ && rm -f skaffold \
      && curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash \
      && install kustomize /usr/local/bin/ && rm -f kustomize /go/kustomize \
      && echo "=== kubectl version ===" && kubectl version --client \
      && echo "=== kustomize version ===" && kustomize version \
      && echo "=== skaffold version ===" && skaffold version \
      \
      #--------------------------------------
      # Krew, kubens, kubectx
      #--------------------------------------
      && ( \
      set -x; cd "$(mktemp -d)" && \
      curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/krew.{tar.gz,yaml}" && \
      tar zxvf krew.tar.gz && \
      KREW=./krew-"$(uname | tr '[:upper:]' '[:lower:]')_amd64" && \
      "$KREW" install --manifest=krew.yaml --archive=krew.tar.gz && \
      "$KREW" update \
      ) \
      && echo 'export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"' >> /root/.bashrc \
      && PATH="${HOME}/.krew/bin:$PATH" kubectl krew install ns \
      && PATH="${HOME}/.krew/bin:$PATH" kubectl krew install ctx \
      \
      #--------------------------------------
      # Okteto CLI
      #--------------------------------------
      && curl https://get.okteto.com -sSfL | sh \
    ; fi \
    \
    #--------------------------------------
    # Minikube
    # Install Minikube because Cloud Code needs even if it is not used.
    # For more detail, see https://github.com/GoogleCloudPlatform/cloud-code-vscode/issues/280
    #--------------------------------------
    && if [ "x${INST_MINIKUBE}" != "x" ] ; then \
      true \
      && curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
      && chmod +x minikube \
      && install minikube /usr/local/bin/ \
      && rm -f minikube \
      && minikube version \
    ; fi \
    \
    #--------------------------------------
    # Istio ctl
    #--------------------------------------
    && ( \
      cd /tmp && curl -L https://istio.io/downloadIstio | sh - \
      && install /tmp/istio-${ISTIO_VERSION}/bin/istioctl /usr/bin \
      && rm -rf /tmp/istio-${ISTIO_VERSION} \
    ) \
    \
    #--------------------------------------
    # Linkerd
    #--------------------------------------
    && curl -sL https://run.linkerd.io/install | \
      LINKERD2_VERSION=${LINKERD2_VERSION} INSTALLROOT=/usr/local sh \
    \
    #--------------------------------------
    # FluxCD
    #--------------------------------------
    && curl -s https://toolkit.fluxcd.io/install.sh | bash \
    \
    #--------------------------------------
    # ArgoCD CLI
    #--------------------------------------
    && curl -sSL -o /tmp/argocd https://github.com/argoproj/argo-cd/releases/download/v${ARGOCD_VERSION}/argocd-linux-amd64 \
    && install /tmp/argocd /usr/local/bin && rm -f /tmp/argocd \
    \
    #--------------------------------------
    # Helm
    #--------------------------------------
    && curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | \
      DESIRED_VERSION=v${HELM_VERSION} bash \
    && helm repo add stable https://charts.helm.sh/stable \
    \
    #--------------------------------------
    # KNative Client
    #--------------------------------------
    && curl -sSL -o /tmp/kn https://github.com/knative/client/releases/download/v${KN_VERSION}/kn-linux-amd64 \
    && install /tmp/kn /usr/local/bin && rm -f /tmp/kn \
    \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=
