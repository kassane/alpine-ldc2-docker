# alpine-ldc2-docker

**DockerHub:** https://hub.docker.com/u/kassany

Building LDC2 (LLVM D Compiler) upstream

#### Tags = version

- `latest`: `master`
- `1.37.0`: `1.37.0`

```bash
## alpine latest (x86_64|ARM64)
docker run --rm -it -v $(pwd):/app -w /app kassany/alpine-ldc2:{tagname} ash
```