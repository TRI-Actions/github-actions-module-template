FROM ubuntu:22.04

# Needed for pipeline
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Install general dependencies
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt install -y \
        # Needed for pre-commit in next build stage
        git \
        libpcre2-8-0 \
        # Builder deps
        unzip \
        software-properties-common \
        # stupid pipeline fix
        sudo \
        curl \
        vim \
        # For terraform tooling
        python3 \
        python3-pip \
        # For jenkins extensions
        default-jre \
        # attempt to fix lc bug
        locales && \
    locale-gen en_US.UTF-8 && \
    # Upgrade pip for be able get latest Checkov
    python3 -m pip install --upgrade pip && \
    # Cleanup
    rm -rf /var/lib/apt/lists/*

# Jenkins lib wants to run as sudo
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME -s /bin/bash \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

ARG PRE_COMMIT_VERSION=${PRE_COMMIT_VERSION:-latest}
ARG TERRAFORM_VERSION=${TERRAFORM_VERSION:-latest}

# Install pre-commit
RUN [ ${PRE_COMMIT_VERSION} = "latest" ] && pip3 install --no-cache-dir pre-commit \
    || pip3 install --no-cache-dir pre-commit==${PRE_COMMIT_VERSION}

# Install terraform because pre-commit needs it
RUN curl -fsSL https://apt.releases.hashicorp.com/gpg | apt-key add - && \
    apt-add-repository "deb https://apt.releases.hashicorp.com $(lsb_release -cs) main" && \
    apt-get update && \
    ( \
        [ "$TERRAFORM_VERSION" = "latest" ] && apt-get install -y terraform \
        || apt-get install -y terraform=${TERRAFORM_VERSION} \
    ) && \
    # Cleanup
    rm -rf /var/lib/apt/lists/*

#
# Install tools
#
WORKDIR /usr/bin

ARG CHECKOV_VERSION=${CHECKOV_VERSION:-false}
ARG TERRAFORM_DOCS_VERSION=${TERRAFORM_DOCS_VERSION:-false}
ARG TERRASCAN_VERSION=${TERRASCAN_VERSION:-false}
ARG TFLINT_VERSION=${TFLINT_VERSION:-false}
ARG TFSEC_VERSION=${TFSEC_VERSION:-false}


# Tricky thing to install all tools by set only one arg.
# In RUN command below used `. /.env` <- this is sourcing vars that
# specified in step below
ARG INSTALL_ALL=${INSTALL_ALL:-false}
RUN if [ "$INSTALL_ALL" != "false" ]; then \
        for variable_name in CHECKOV_VERSION TERRAFORM_DOCS_VERSION TERRASCAN_VERSION TFLINT_VERSION TFSEC_VERSION; do \
            variable_value="$(eval "echo \$$variable_name")" \
            ; if [ "${variable_value}" = "false" -o -z "${variable_value}" ]; then \
                echo "export ${variable_name}=latest" >> /.env \
            ; else \
                echo "export ${variable_name}=${variable_value}" >> /.env \
            ; fi \
        ; done \
    ; else \
        echo "" >> /.env \
    ; fi


# Checkov
RUN . /.env && \
    if [ "$CHECKOV_VERSION" != "false" ]; then \
    ( \
        [ "$CHECKOV_VERSION" = "latest" ] && pip3 install --no-cache-dir checkov \
        || pip3 install --no-cache-dir checkov==${CHECKOV_VERSION} \
    ) \
    ; fi

# Terraform docs
RUN . /.env && \
    if [ "$TERRAFORM_DOCS_VERSION" != "false" ]; then \
    ( \
        TERRAFORM_DOCS_RELEASES="https://api.github.com/repos/terraform-docs/terraform-docs/releases" && \
        [ "$TERRAFORM_DOCS_VERSION" = "latest" ] && curl -L "$(curl -s ${TERRAFORM_DOCS_RELEASES}/latest | grep -o -E -m 1 "https://.+?-linux-amd64.tar.gz")" > terraform-docs.tgz \
        || curl -L "$(curl -s ${TERRAFORM_DOCS_RELEASES} | grep -o -E "https://.+?v${TERRAFORM_DOCS_VERSION}-linux-amd64.tar.gz")" > terraform-docs.tgz \
    ) && tar -xzf terraform-docs.tgz terraform-docs && rm terraform-docs.tgz && chmod +x terraform-docs \
    ; fi

# Terrascan
RUN . /.env && \
    if [ "$TERRASCAN_VERSION" != "false" ]; then \
    ( \
        TERRASCAN_RELEASES="https://api.github.com/repositories/103084166/releases" && \
        [ "$TERRASCAN_VERSION" = "latest" ] && curl -L "$(curl -s ${TERRASCAN_RELEASES}/latest | grep -o -E -m 1 "https://.+?_Linux_x86_64.tar.gz")" > terrascan.tar.gz \
        || curl -L "$(curl -s ${TERRASCAN_RELEASES} | grep -o -E "https://.+?${TERRASCAN_VERSION}_Linux_x86_64.tar.gz")" > terrascan.tar.gz \
    ) && tar -xzf terrascan.tar.gz terrascan && rm terrascan.tar.gz && \
    ./terrascan init \
    ; fi

# TFLint
RUN . /.env && \
    if [ "$TFLINT_VERSION" != "false" ]; then \
    ( \
        TFLINT_RELEASES="https://api.github.com/repos/terraform-linters/tflint/releases" && \
        [ "$TFLINT_VERSION" = "latest" ] && curl -L "$(curl -s ${TFLINT_RELEASES}/latest | grep -o -E -m 1 "https://.+?_linux_amd64.zip")" > tflint.zip \
        || curl -L "$(curl -s ${TFLINT_RELEASES} | grep -o -E "https://.+?/v${TFLINT_VERSION}/tflint_linux_amd64.zip")" > tflint.zip \
    ) && unzip tflint.zip && rm tflint.zip \
    ; fi

# TFSec
RUN . /.env && \
    if [ "$TFSEC_VERSION" != "false" ]; then \
    ( \
        TFSEC_RELEASES="https://api.github.com/repos/aquasecurity/tfsec/releases" && \
        [ "$TFSEC_VERSION" = "latest" ] && curl -L "$(curl -s ${TFSEC_RELEASES}/latest | grep -o -E -m 1 "https://.+?/tfsec-linux-amd64")" > tfsec \
        || curl -L "$(curl -s ${TFSEC_RELEASES} | grep -o -E -m 1 "https://.+?v${TFSEC_VERSION}/tfsec-linux-amd64")" > tfsec \
    ) && chmod +x tfsec \
    ; fi

# Checking binaries versions and write it to debug file
RUN . /.env && \
    F=tools_versions_info && \
    pre-commit --version >> $F && \
    terraform --version | head -n 1 >> $F && \
    (if [ "$CHECKOV_VERSION"        != "false" ]; then echo "checkov $(checkov --version)" >> $F;     else echo "checkov SKIPPED" >> $F       ; fi) && \
    (if [ "$TERRAFORM_DOCS_VERSION" != "false" ]; then ./terraform-docs --version >> $F;              else echo "terraform-docs SKIPPED" >> $F; fi) && \
    (if [ "$TERRASCAN_VERSION"      != "false" ]; then echo "terrascan $(./terrascan version)" >> $F; else echo "terrascan SKIPPED" >> $F     ; fi) && \
    (if [ "$TFLINT_VERSION"         != "false" ]; then ./tflint --version >> $F;                      else echo "tflint SKIPPED" >> $F        ; fi) && \
    (if [ "$TFSEC_VERSION"          != "false" ]; then echo "tfsec $(./tfsec --version)" >> $F;       else echo "tfsec SKIPPED" >> $F         ; fi) && \
    echo "\n\n" && cat $F && echo "\n\n"

ENV PRE_COMMIT_COLOR=${PRE_COMMIT_COLOR:-always}