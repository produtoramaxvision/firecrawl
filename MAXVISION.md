# MaxVision fork of Firecrawl

This fork runs the self-hosted Firecrawl of Produtora MaxVision (Portainer stack
`firecrawl_mxv` on the Oracle ARM64 VPS). Branch `mxv/self-host` is upstream
`firecrawl/firecrawl` plus the patches below, one commit each, so every patch can be
re-applied on a newer upstream commit and offered upstream on its own.

## Patches

1. `feat(api): make the self-hosted concurrency limit configurable`
   Upstream falls back to a hardcoded limit of 2 when Autumn billing is not
   configured, which is every self-hosted deployment, and all self-hosted requests
   share one team. `DEFAULT_CONCURRENCY_LIMIT` (default 2) sets that fallback.

## Build (on the VPS, arm64)

```sh
git clone --depth 1 --branch mxv/self-host https://github.com/produtoramaxvision/firecrawl.git
cd firecrawl/apps/api
SHA=$(git rev-parse HEAD)
docker buildx build --load --platform linux/arm64 --build-arg GIT_SHA=$SHA -t firecrawl-mxv:${SHA:0:8} .
docker builder prune -f   # the Rust/pnpm build cache is several GB
```

Check `df -h /` before and after: the Swarm raft log has been corrupted by a full disk
before. Then point `x-firecrawl-image` in the stack compose at the new tag.

## Updating to a newer upstream

Rebase `mxv/self-host` on the new upstream commit, rebuild, and re-run the checks in
the stack: scrape (markdown, json), crawl, batch, extract, deep research, llms.txt and
`/v2/team/queue-status` (maxConcurrency must equal `DEFAULT_CONCURRENCY_LIMIT`).
