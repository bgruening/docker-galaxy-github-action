# Build and push Galaxy NGS image (composite action)

Composite GitHub Action that builds the Galaxy NGS preprocessing container from this repo and optionally pushes it to a registry. The action wires up the build args for the tools YAML and the base Galaxy image so they can be overridden per workflow.

## Inputs
- `repository` (required): Image repository, e.g. `username/galaxy-ngs`.
- `registry` (default `docker.io`): Registry hostname (e.g. `ghcr.io`).
- `tags` (default `latest`): Tags to apply, comma or newline separated.
- `tool-file` (default `ngs_preprocessing.yml`): Tools YAML passed as build arg `TOOL_FILE`.
- `base-image` (default `quay.io/bgruening/galaxy:25.1.1`): Base Galaxy image passed as build arg `BASE_IMAGE`.
- `context` (default `.`): Docker build context.
- `dockerfile` (default `Dockerfile`): Path to the Dockerfile.
- `build-args` (optional): Extra build args (`KEY=VALUE`, comma or newline separated).
- `push` (default `true`): Push to the registry. When `false`, the image is loaded locally (single platform builds only).
- `username` / `password`: Registry credentials (required when `push=true`).
- `provenance` (default `false`): Enable provenance attestation if the builder supports it.

## Usage
Example workflow job that builds and pushes to Docker Hub:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build and push
        uses: ./action
        with:
          registry: docker.io
          repository: myuser/galaxy-ngs
          tags: |
            latest
            ${{ github.sha }}
          tool-file: ngs_preprocessing.yml
          base-image: quay.io/bgruening/galaxy:25.1.1
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
```

Notes:
- You can override `tool-file` to point at an alternate tools YAML or supply additional build args via `build-args` as needed.
- To match container tags to GitHub releases, add `${{ github.event.release.tag_name }}` (for release events) or `${{ github.ref_name }}` (for tag workflows) to `tags`, e.g.:

  ```yaml
  tags: |
    latest
    ${{ github.event.release.tag_name }}
  ```
