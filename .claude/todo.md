# TODO

## Open

- [ ] **Fix or remove the non-amd64 Dockerfiles' qemu builder stage.** Found 2026-11-15 while building v2.1.
  - **Symptom:** `arm64v8`, `armv7`, `ppc64le`, `riscv64` and `s390x` fail to build with `Package 'qemu-user-static' has no installation candidate`. `amd64.Dockerfile` is unaffected only because BuildKit skips its unused builder stage.
  - **Cause:** since QEMU 9.1, Debian and Ubuntu fold `qemu-user-static` into `qemu-user`. Ubuntu 24.04 still has the old package; 26.04 and `rolling` (26.10) do not. The binaries are still statically linked (static-pie), but they no longer carry the `-static` suffix (`/usr/bin/qemu-aarch64`, not `/usr/bin/qemu-aarch64-static`).
  - **Scope:** only this repo. The pattern (install qemu in a builder stage, `COPY` the binary into the image) and `hooks/pre_build`'s `multiarch/qemu-user-static:register` both come from upstream TheRemote and predate binfmt_misc's fix-binary (`F`) flag. Every other repo registers emulation on the build host instead: `docker/setup-qemu-action` on GitHub, and the GitLab runner's `skynet/binfmt` (`tonistiigi/binfmt`) pre-build step.
  - **Options:**
    1. Delete the `builder` stage and the `COPY --from=builder /usr/bin/qemu-*-static` line from each non-amd64 Dockerfile, and rely on host binfmt (buildx and Docker Desktop already provide it).
    2. Delete the non-amd64 Dockerfiles, `hooks/` and `multi-arch-manifest.yaml`, since only amd64 is published (from `dev/publish.sh`).
    3. Keep the stage but install `qemu-user` and copy `/usr/bin/qemu-<arch>`.
- [ ] **`dev/test.sh` still checks Paper's old `api.papermc.io` v2 API**, while `start.sh` uses `fill.papermc.io` v3 (noted in `CLAUDE.md`).
