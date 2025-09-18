FROM container-registry.oracle.com/os/oraclelinux:9 AS builder

RUN dnf config-manager --enable ol9_codeready_builder && \
    dnf install -y oracle-ocne-release-el9 && \
    dnf config-manager --enable ol9_olcne19 && \
    dnf -y install git make golang hostname yq && \
    go version

# Copy the contents of the repository
COPY . /workspace/go/src/github.com/oracle-cne/promu

ENV GOPATH=/workspace/go CGO_ENABLED=0
ENV GOPATH_SRC=$GOPATH/src/github.com/oracle-cne/promu
WORKDIR /workspace/go/src/github.com/oracle-cne/promu

RUN git config --global --add safe.directory $(pwd) && \
    yq -i '.build.flags = "-trimpath=false"' .promu.yml && \
    yq -i '.build.ldflags += "-X main.version=v${.Version}"' .promu.yml && \
    sed -i 's|github.com/prometheus/promu||g' Makefile && \
    make build

FROM container-registry.oracle.com/os/oraclelinux:9-slim

COPY LICENSE THIRD_PARTY_LICENSES.txt olm/SECURITY.md /usr/share/licenses/promu/
COPY --from=builder /workspace/go/src/github.com/oracle-cne/promu /usr/bin/promu

RUN microdnf update -y &&\
    microdnf clean all &&\
    test -d /LICENSES || ln -s /usr/share/licenses /LICENSES

USER nobody

ENTRYPOINT [ "/usr/bin/promu" ]
