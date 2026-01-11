# Build and push Galaxy Docker flavors via GitHub actions

GitHub Action that builds the Galaxy Docker flavors and optionally pushes it to a Docker registry.

## Inputs
- `repository` (required): Image repository, e.g. `username/galaxy-ngs`.
- `registry` (default `docker.io`): Registry hostname (e.g. `ghcr.io` or `quay.io`).
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
env:
  IMAGE_REGISTRY: quay.io
  IMAGE_REPOSITORY: bgruening/galaxy-ngs-preprocessing
  GALAXY_BASE_IMAGE: quay.io/bgruening/galaxy:25.1
  TOOL_FILE: ngs_preprocessing.yml

jobs:
  build:
    if: github.event_name != 'release'
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Build image (no push)
        uses: bgruening/docker-galaxy-github-action@v1
        with:
          registry: ${{ env.IMAGE_REGISTRY }}
          repository: ${{ env.IMAGE_REPOSITORY }}
          tags: ci-${{ github.sha }}
          tool-file: ${{ env.TOOL_FILE }}
          base-image: ${{ env.GALAXY_BASE_IMAGE }}
          push: false

  release:
    if: github.event_name == 'release'
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Build and push image
        uses: bgruening/docker-galaxy-github-action@v1
        with:
          registry: ${{ env.IMAGE_REGISTRY }}
          repository: ${{ env.IMAGE_REPOSITORY }}
          push: true
          tool-file: ${{ env.TOOL_FILE }}
          base-image: ${{ env.GALAXY_BASE_IMAGE }}
          username: '$oauthtoken'
          password: ${{ secrets.QUAY_OAUTH_TOKEN }}
          tags: |
            ${{ github.event.release.tag_name }}
            latest
```

Notes:
- You can override `tool-file` to point at an alternate tools YAML or supply additional build args via `build-args` as needed.
- To match container tags to GitHub releases, add `${{ github.event.release.tag_name }}` (for release events) or `${{ github.ref_name }}` (for tag workflows) to `tags`, e.g.:

  ```yaml
  tags: |
    latest
    ${{ github.event.release.tag_name }}
  ```
