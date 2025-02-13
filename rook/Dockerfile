# Rook container

# Stage1: build from source
FROM quay.io/cybozu/golang:1.16-focal AS build

ARG ROOK_VERSION=1.7.5
ARG ROOK_DIR=/work/go/src/github.com/rook/rook

SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV GOPATH=/work/go
RUN git clone https://github.com/rook/rook.git ${ROOK_DIR}

COPY expand-timeout.patch /tmp/

WORKDIR ${ROOK_DIR}

RUN git checkout v${ROOK_VERSION}
RUN patch -p1 --no-backup-if-mismatch < /tmp/expand-timeout.patch
RUN make build IMAGES="ceph" BUILD_CONTAINER_IMAGE=false

WORKDIR ${ROOK_DIR}/images/ceph
RUN mkdir /tmp/csv_template_dir
RUN make CSV_TEMPLATE_DIR=/tmp/csv_template_dir generate-csv-templates

# Stage2: setup runtime container
FROM quay.io/cybozu/ceph:16.2.6.0

ARG TINI_VERSION=v0.19.0
ARG ROOK_DIR=/work/go/src/github.com/rook/rook

COPY --from=build ${ROOK_DIR}/_output/bin/linux_amd64/rook \
    ${ROOK_DIR}/_output/bin/linux_amd64/rookflex \
    ${ROOK_DIR}/images/ceph/toolbox.sh \
    ${ROOK_DIR}/images/ceph/set-ceph-debug-level \
    /usr/local/bin/

COPY --from=build ${ROOK_DIR}/cluster/examples/kubernetes/ceph/csi/template /etc/ceph-csi
COPY --from=build ${ROOK_DIR}/cluster/examples/kubernetes/ceph/monitoring /etc/ceph-monitoring
COPY --from=build ${ROOK_DIR}/cluster/examples/kubernetes/ceph/create-external-cluster-resources.* /etc/rook-external/
COPY --from=build ${ROOK_DIR}/cluster/examples/kubernetes/ceph/test-data/ceph-status-out /etc/rook-external/test-data/
COPY --from=build /tmp/csv_template_dir/cluster/olm/ceph/templates /etc/ceph-csv-templates
COPY --from=build ${ROOK_DIR}/LICENSE /usr/local/rook/LICENSE

ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-static-amd64 /tini
RUN chmod 755 /tini

# create or modify owner and permissions to make a watch-active container of a MGR work properly
RUN install -d -o 10000 -g 10000 -m 0700 /var/lib/rook
RUN chown 10000:10000 /etc/ceph

USER 10000:10000

ENTRYPOINT ["/tini", "--", "/usr/local/bin/rook"]
