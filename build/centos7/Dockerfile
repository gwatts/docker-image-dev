# Copyright Verizon Media. Licensed under the terms of the Apache 2.0 license. See LICENSE in the project root.
FROM centos:7

# Needed to build vespa
RUN yum-config-manager --add-repo https://copr.fedorainfracloud.org/coprs/g/vespa/vespa/repo/epel-7/group_vespa-vespa-epel-7.repo && \
    yum -y install epel-release && \
    yum -y install centos-release-scl && \
    yum -y --setopt=skip_missing_names_on_install=False --enablerepo=epel-testing install \
        git \
        jq \
        yum-utils \
        ccache \
        sudo

ENV GIT_REPO "https://github.com/vespa-engine/vespa.git"

# Change git reference for a specific version of the vespa.spec file. Use a tag or SHA to allow for reproducible builds.
ENV VESPA_SRC_REF "29249eef05f2d29a8bb47aa9d5a7fa68f000c3c2"

# Install vespa build and runtime dependencies
RUN git clone $GIT_REPO && cd vespa && git -c advice.detachedHead=false checkout $VESPA_SRC_REF && \
    sed -e '/^BuildRequires:/d' -e '/^Requires: %{name}-/d' -e 's/^Requires:/BuildRequires:/' dist/vespa.spec > dist/vesparun.spec && \
    yum-builddep -y --setopt="centos-sclo-rh-source.skip_if_unavailable=true" dist/vespa.spec dist/vesparun.spec && \
    cd .. && rm -r vespa && \
    alternatives --set java java-11-openjdk.x86_64 && \
    alternatives --set javac java-11-openjdk.x86_64 && \
    yum clean all && rm -rf /var/cache/yum && \
    echo -e "#!/bin/bash\nsource /opt/rh/devtoolset-9/enable" >> /etc/profile.d/enable-devtoolset-9.sh && \
    echo -e "#!/bin/bash\nsource /opt/rh/rh-maven35/enable" >> /etc/profile.d/enable-rh-maven35.sh && \
    echo -e "* soft nproc 409600\n* hard nproc 409600" > /etc/security/limits.d/99-nproc.conf && \
    echo -e "* soft nofile 262144\n* hard nofile 262144" > /etc/security/limits.d/99-nofile.conf

# Java requires proper locale for unicode
ENV LANG en_US.UTF-8
