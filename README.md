# alpine-ldc2-docker

**DockerHub:** https://hub.docker.com/u/kassany

Building LDC2 (LLVM D Compiler) and OpenD-LDC upstream

#### Tags = version

- `latest`: `master`
- `1.41.0`: `v1.41.0` **ldc2 only**
- `1.40.0`: `v1.40.0` **ldc2 only**
- `1.39.0`: `v1.39.0` **ldc2 only**

```bash
## alpine latest (x86_64|ARM64)
docker run --rm -it -v $(pwd):/app -w /app kassany/alpine-ldc2:{tagname} ash
# or
docker run --rm -it -v $(pwd):/app -w /app kassany/alpine-opend:{tagname} ash
```