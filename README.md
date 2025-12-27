# soarpkgs Test Playground

Test repository for validating the soarpkgs build system before deploying to production.

## Structure

```
soarpkgs-test/
├── .github/workflows/
│   ├── matrix_builds.yaml      # Reusable build workflow (from soarpkgs)
│   ├── build-on-change.yaml    # Trigger on recipe changes
│   └── manual-build.yaml       # Manual trigger for testing
├── binaries/                   # Static binaries -> bincache
│   ├── hello/
│   │   └── static.yaml         # Simple C binary (fast build)
│   └── jq/
│       └── static.yaml         # Download pre-built jq
└── packages/                   # AppImages, archives -> pkgcache
    └── hello-appimage/
        └── appimage.yaml       # Simple AppImage
```

## Quick Start

1. Create a new GitHub repository (e.g., `pkgforge/soarpkgs-test`)

2. Copy this directory to the new repo:
   ```bash
   cd soarpkgs-test
   git init
   git add .
   git commit -m "Initial test repo"
   git remote add origin git@github.com:pkgforge/soarpkgs-test.git
   git push -u origin main
   ```

3. Set up required secrets in the new repo:
   - `RO_GHTOKEN` - Read-only GitHub token for API requests
   - `RO_GLTOKEN` - Read-only GitLab token (optional)
   - `MINISIGN_KEY` - Minisign private key for signing (optional)
   - `MINISIGN_SIGKEY` - Minisign signature key (optional)

4. Go to Actions tab and run workflows

## Workflows

### Manual Build Test
Best for testing individual recipes:
1. Go to Actions → "Manual Build Test"
2. Select a recipe and host
3. Run workflow

### Build on Change
Automatically builds when recipes change:
- Triggered on push to main
- Detects which recipes changed
- Routes to bincache or pkgcache based on path

## Test Recipes

| Recipe | Type | Build Time | Tests |
|--------|------|------------|-------|
| `binaries/hello/static.yaml` | static | ~30s | musl-gcc compilation |
| `binaries/jq/static.yaml` | static | ~10s | Pre-built download |
| `packages/hello-appimage/appimage.yaml` | appimage | ~1m | AppImage creation |

## Cache Routing

- `binaries/**/*.yaml` → `ghcr.io/pkgforge/bincache-test`
- `packages/**/*.yaml` → `ghcr.io/pkgforge/pkgcache-test`

## What Gets Tested

1. **sbuild download** - Fetches from pkgforge/sbuilder releases
2. **sbuild info** - Recipe parsing and host compatibility
3. **sbuild build** - Full package build with:
   - Checksum generation
   - GHCR push
   - Optional signing
4. **bincache/pkgcache separation** - Correct routing based on path
5. **Matrix builds** - Multi-arch builds (x86_64, aarch64)
