# Changes Log

## 2026-11-04 10:20:58 -05:00
Scope: IIIF/OpenSeadragon stability, embed/build consistency, and smoke-test cleanup.

### Added
- New load-session lifecycle controls in OpenSeadragon page loading (`beginLoadSession`, `isCurrentLoadSession`, `endLoadSession`) to prevent stale async completion races.
  - File: src/content-handlers/iiif/modules/uv-openseadragoncenterpanel-module/OpenSeadragonCenterPanel.ts
- Fail-safe `openMedia` timeout and failure reporting events for load failure scenarios.
  - File: src/content-handlers/iiif/modules/uv-openseadragoncenterpanel-module/OpenSeadragonCenterPanel.ts
- Retry/backoff JSON fetch helper for Girder tile descriptor loading.
  - File: src/content-handlers/iiif/modules/uv-openseadragoncenterpanel-module/OpenSeadragonCenterPanel.ts
- Canvas-based main-image fallback extraction methods (`getImageResourcesFromCanvases`, `getCanvasBodyImageUri`) for missing/thumbnail-biased external resources.
  - File: src/content-handlers/iiif/modules/uv-openseadragoncenterpanel-module/OpenSeadragonCenterPanel.ts
- New smoke script for automated paging stall detection with env-based runtime options.
  - File: scripts/smoke-osd-load.js
- New webpack-specific TypeScript config.
  - File: tsconfig.webpack.json

### Changed
- Updated image URI resolution so main image rendering prefers canvas/body image URLs and can recover from thumbnail URLs (`op=thumb` to `op=img` fallback as last resort).
  - File: src/content-handlers/iiif/modules/uv-openseadragoncenterpanel-module/OpenSeadragonCenterPanel.ts
- Updated OSD runtime defaults:
  - `imageLoaderLimit: 8`
  - `maxImageCacheCount: 200`
  - `openMediaTimeout: 20000`
  - File: src/content-handlers/iiif/extensions/uv-openseadragon-extension/config/config.json
- Updated config template URL to HTTPS.
  - File: src/content-handlers/iiif/extensions/uv-openseadragon-extension/config/config.json
- Updated embed template script loading to use template-injected head tags and removed hardcoded script source.
  - File: src/uv.html
- Updated dev template injection behavior to avoid duplicate/misplaced script tags (`inject: false`).
  - File: webpack.dev-server.js
- Updated production copy transforms to inject `umd/UV.js` into copied templates at build time.
  - File: webpack.config.js
- Updated TypeScript compiler options (added `ignoreDeprecations: "6.0"`; removed deprecated `suppressImplicitAnyIndexErrors`).
  - File: tsconfig.json
- Updated package/dependency alignment required during stabilization and build compatibility work.
  - File: package.json

### Fixed
- Fixed intermittent fullscreen paging stalls caused by async race conditions and incomplete load-completion paths.
  - File: src/content-handlers/iiif/modules/uv-openseadragoncenterpanel-module/OpenSeadragonCenterPanel.ts
- Fixed cases where main canvas failed to request full image endpoints and only thumbnails were requested.
  - File: src/content-handlers/iiif/modules/uv-openseadragoncenterpanel-module/OpenSeadragonCenterPanel.ts
- Fixed invalid viewport/bounds operations when no items were loaded (`updateBounds`, `goHome` guards), addressing zero-height assertion behavior.
  - File: src/content-handlers/iiif/modules/uv-openseadragoncenterpanel-module/OpenSeadragonCenterPanel.ts
- Fixed dev/prod embed script mismatch that could produce MIME/path issues for the UV bundle.
  - Files: src/uv.html, webpack.dev-server.js, webpack.config.js
- Fixed smoke test verbosity by removing temporary telemetry/debug logging in cleanup passes while preserving pass/fail checks.
  - File: scripts/smoke-osd-load.js

### Result
- Paging in fullscreen is stable under tested long-manifest scenarios.
- Main viewer requests full image endpoints when available/recoverable.
- Dev and production embed pages consistently load the correct viewer bundle.

## 2026-11-04 11:34:00 -05:00
Scope: Additional resilience hardening for embed config loading and remote tile descriptor fetches.

### Added
- Config host allowlist support in embed mode (`window.UV_ALLOWED_CONFIG_HOSTS`) for safer remote configuration.
  - File: src/uv.html
- Config shape validation before applying remote configuration payloads.
  - File: src/uv.html
- Retry with exponential backoff + jitter for embed config fetch (`fetchConfigWithRetry`).
  - File: src/uv.html
- Generic async wait and retrying JSON fetch helper methods for OSD remote requests (`wait`, `fetchJsonWithRetry`).
  - File: src/content-handlers/iiif/modules/uv-openseadragoncenterpanel-module/OpenSeadragonCenterPanel.ts

### Changed
- Hardened remote config URL checks to require protocol + allowed host validation (including controlled subdomain matching).
  - File: src/uv.html
- Updated Girder tile descriptor loading to use bounded retry/timeout handling and explicit failure reporting.
  - File: src/content-handlers/iiif/modules/uv-openseadragoncenterpanel-module/OpenSeadragonCenterPanel.ts

### Fixed
- Reduced risk of applying malformed remote config data by rejecting invalid config shapes and falling back safely.
  - File: src/uv.html
- Improved resilience to transient network failures when loading Girder tile metadata.
  - File: src/content-handlers/iiif/modules/uv-openseadragoncenterpanel-module/OpenSeadragonCenterPanel.ts

### Result
- Embed configuration loading is safer and more fault-tolerant.
- Remote tile descriptor acquisition recovers better from intermittent failures.

## 2026-11-04 12:05:00 -05:00
Scope: Build pipeline compatibility and compact production output.

### Added
- Build-specific TypeScript settings are now explicitly used by webpack via `ts-loader` `configFile` mapping.
  - File: webpack.config.js

### Changed
- Updated webpack TypeScript loader rules to use `tsconfig.webpack.json` for production builds.
  - File: webpack.config.js
- Updated production build output for smaller artifacts:
  - Disabled source map emission (`devtool: false`).
  - Shortened dynamic chunk names (`chunkFilename: "[id].js"`).
  - Enabled export mangling tuned for size (`mangleExports: "size"`).
  - File: webpack.config.js
- Updated webpack TypeScript build profile to emit more compact JS (`removeComments: true`, `sourceMap: false`).
  - File: tsconfig.webpack.json
- Disabled webpack performance hints in production output to suppress non-blocking size warnings.
  - File: webpack.config.js

### Fixed
- Fixed production build failure caused by unsupported TypeScript compiler options in the existing `ts-loader`/TypeScript 4.x build path.
  - Files: tsconfig.webpack.json, webpack.config.js
- Fixed noisy/non-actionable build warning output in CI/local tasks by removing performance-hint warnings.
  - File: webpack.config.js

### Result
- `npm run build` completes successfully with no warnings.
- Production build artifacts are emitted in a more compact form.
