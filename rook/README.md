Rook container
==============

This container uses a [rook][].

`expand-timeout.patch` resolves the issue that trying run of radosgw-admin is timeout.
After resolved the issue in the upstream, remove the patch.

`fix-liveness-probe.path` resolves the wrong liveness probe command when configure liveness probe.

[rook]: https://github.com/rook/rook

Docker images
-------------

Docker images are available on [Quay.io](https://quay.io/repository/cybozu/rook)
