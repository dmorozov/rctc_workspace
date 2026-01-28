# Ubuntu slim base with JetBrains Runtime (JBR) DCEVM for Java hot-swap
FROM ubuntu:24.04

# Avoid prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Install base dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    wget \
    curl \
    ca-certificates \
    gnupg \
    lsb-release \
    sudo \
    git \
    gh \
    subversion \
    libapache2-mod-svn \
    unzip \
    xz-utils \
    fontconfig \
    locales \
    && rm -rf /var/lib/apt/lists/* \
    && locale-gen en_US.UTF-8

ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US:en
ENV LC_ALL=en_US.UTF-8

# Create vscode user (same as devcontainers base image)
ARG USERNAME=ubuntu
ARG USER_GID=ubuntu

RUN echo "$USERNAME ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# ============================================
# Install JetBrains Runtime (JBR) with DCEVM
# ============================================
# JBR includes DCEVM patches for enhanced class redefinition
# and built-in HotswapAgent support
ARG JBR_VERSION=21.0.5
ARG JBR_BUILD=b631.16
ENV JAVA_HOME=/opt/jbr-21
ENV PATH=$JAVA_HOME/bin:$PATH

RUN wget -q https://cache-redirector.jetbrains.com/intellij-jbr/jbr-${JBR_VERSION}-linux-x64-${JBR_BUILD}.tar.gz -O /tmp/jbr.tar.gz \
    && mkdir -p /opt \
    && tar -xzf /tmp/jbr.tar.gz -C /opt \
    && mv /opt/jbr-${JBR_VERSION}-linux-x64-${JBR_BUILD} $JAVA_HOME \
    && rm /tmp/jbr.tar.gz \
    && echo "JBR installed: $(java -version 2>&1 | head -1)"

# Download HotswapAgent and place in JBR lib/hotswap
ARG HOTSWAP_AGENT_VERSION=2.0.1
RUN mkdir -p $JAVA_HOME/lib/hotswap \
    && wget -q https://github.com/HotswapProjects/HotswapAgent/releases/download/RELEASE-${HOTSWAP_AGENT_VERSION}/hotswap-agent-${HOTSWAP_AGENT_VERSION}.jar \
        -O $JAVA_HOME/lib/hotswap/hotswap-agent.jar

# ============================================
# Install Maven
# ============================================
ARG MAVEN_VERSION=3.9.12
RUN wget -q https://dlcdn.apache.org/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz -O /tmp/maven.tar.gz \
    && tar -xzf /tmp/maven.tar.gz -C /opt \
    && ln -s /opt/apache-maven-${MAVEN_VERSION}/bin/mvn /usr/local/bin/mvn \
    && rm /tmp/maven.tar.gz

# Install Node.js
ARG NODE_VERSION=24.12.0
ENV PATH="/opt/node/bin:${PATH}"
RUN ARCH=$(dpkg --print-architecture) \
    && if [ "$ARCH" = "amd64" ]; then NODE_ARCH="x64"; else NODE_ARCH="arm64"; fi \
    && curl -fsSL "https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-${NODE_ARCH}.tar.xz" -o /tmp/node.tar.xz \
    && mkdir -p /opt/node \
    && tar -xJf /tmp/node.tar.xz -C /opt/node --strip-components=1 \
    && rm /tmp/node.tar.xz \
    && node --version \
    && npm --version

# ============================================
# TomEE Setup (external server for RSP connector)
# ============================================
ENV CATALINA_HOME=/opt/tomcat
ENV PATH=$CATALINA_HOME/bin:$PATH
RUN mkdir -p $CATALINA_HOME && chown $USERNAME:$USERNAME $CATALINA_HOME

# Set ownership for ubuntu user
RUN chown -R ${USERNAME}:${USER_GID} /opt/tomcat /opt/node
# Configure npm to use a user-writable directory for global packages
RUN mkdir -p /home/${USERNAME}/.npm-global \
    && chown -R ${USERNAME}:${USER_GID} /home/${USERNAME}/.npm-global \
    && echo "prefix=/home/${USERNAME}/.npm-global" > /home/${USERNAME}/.npmrc \
    && chown ${USERNAME}:${USER_GID} /home/${USERNAME}/.npmrc

# Add npm global bin to PATH for all users
ENV PATH="/home/${USERNAME}/.npm-global/bin:${PATH}"

USER $USERNAME

# Download TomEE Plus
WORKDIR /tmp
RUN wget -q https://dlcdn.apache.org/tomee/tomee-10.1.3/apache-tomee-10.1.3-plus.tar.gz -O tomcat.tar.gz \
    && tar xzf tomcat.tar.gz \
    && mv apache-tomee-plus-10.1.3/* $CATALINA_HOME/ \
    && rm -rf apache-tomee-plus-10.1.3 tomcat.tar.gz

# Download DB2 JDBC driver
RUN wget -q https://repo1.maven.org/maven2/com/ibm/db2/jcc/11.5.9.0/jcc-11.5.9.0.jar -P $CATALINA_HOME/lib/
RUN wget -q https://repo1.maven.org/maven2/org/postgresql/postgresql/42.7.9/postgresql-42.7.9.jar -P $CATALINA_HOME/lib/

WORKDIR /workspaces

# Copy TomEE configuration files
COPY --chown=$USERNAME:$USERNAME tomee/tomee.xml ${CATALINA_HOME}/conf/tomee.xml
COPY --chown=$USERNAME:$USERNAME tomee/tomcat-users.xml ${CATALINA_HOME}/conf/tomcat-users.xml
COPY --chown=$USERNAME:$USERNAME tomee/system.properties ${CATALINA_HOME}/conf/system.properties
COPY --chown=$USERNAME:$USERNAME tomee/logging.properties ${CATALINA_HOME}/conf/logging.properties
COPY --chown=$USERNAME:$USERNAME tomee/pdq.jar ${CATALINA_HOME}/lib/pdq.jar

# Setup RSP server connector directory
RUN mkdir -p /home/$USERNAME/.rsp/redhat-community-server-connector/servers
COPY --chown=$USERNAME:$USERNAME tomee/TomEE_10 /home/$USERNAME/.rsp/redhat-community-server-connector/servers/TomEE_10

# Verify installation
RUN java -version && mvn -version && echo "Build complete: JBR DCEVM + Maven + TomEE"

# Set the working directory for the user
WORKDIR /workspaces
