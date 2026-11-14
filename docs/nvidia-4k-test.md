# Optional NVIDIA 4K test — v0.5.7.2

This build includes the Full HD compatibility fix and adds **4K test (NVIDIA, this page only)**. It is an experiment, not verified NVIDIA 4K support. Keep the working v0.5.7.1 package for rollback.

## Try it

1. Extract the v0.5.7.2 ZIP into a separate permanent folder.
2. In `edge://extensions`, disable the previous Disney+ extension. Load this package's **extension** folder with **Load unpacked**. Run only one extension/userscript copy.
3. Reload Disney+, press **Alt+Shift+4**, and select **Full HD** first if needed.
4. Open the panel and select **4K test (NVIDIA, this page only)**. It appears after the panel identifies NVIDIA as the rendering GPU. This does not prove the protected-video output route.
5. After the page reloads, manually start a title labeled 4K on a plan that includes it. Allow adaptive quality to settle. The test requests 3840x2160 without requesting HDR; it does not change Windows HDR settings and does not guarantee SDR delivery.
6. Look at **Video playing**, not the `4K?` request badge. A successful stream needs actual **3840 × 2160**, advancing video time and frames, and sound. The video may remain lower resolution or fail if the service/CDM/output path does not permit 4K. A 4K source can also be downscaled by a lower-resolution screen.
7. Reload to end the experiment and return to the previous Full HD/OFF setting. You can also select **Full HD** directly. If it fails, copy the panel diagnostics rather than repeatedly retrying. Disable the new build and re-enable v0.5.7.1 to roll back.

A driver or system freeze cannot be prevented by a JavaScript timer. The upstream NVIDIA experiments encountered output-protection errors; upstream's successful 4K results were on Intel. Do not repeatedly run a configuration that freezes the PC.

## What changed

- The existing `4k-hevc-sdk-playready` path now accepts the statically inspected **26.15.0-owl** service and session SDKs, in addition to its original release. Other UHD modes do not gain this acceptance.
- The SDK hook changes the per-session resolution list and VOD request ceiling. It validates the exact trusted session script path and requires the service version, session URL version and constructor arguments to agree.
- The user-facing entry checks NVIDIA before selection and again after reload. Its one-use ticket is consumed before that second check; it never persists 4K as the default. Full HD or OFF comes back on the next reload.
- Native decoder/CDM results, license responses, output restrictions, format exclusions and the SDK's DRM preflight remain authoritative. The test does not force a successful DRM result, extract keys or bypass HDCP. It keeps adaptive selection and creates at most one SDK session per experimental page.
- `4K?` means only that an experimental 4K request is active. Actual decoded dimensions and frame progress are reported separately.

## Compatibility review and verification

The public 26.15.0-owl service still constructs `PlaybackSession` with `playbackServiceVersion`, `configManager`, and `mediaCapabilities`. The session still reads `videoResolutions` for variant filtering. The UMD export, per-session settings merge and native DRM flow were inspected in the public bundles already recorded with hashes in [the Full HD investigation](sdk-26.15-compatibility.md). No proprietary SDK files are included in this package.

The offline suite now has **31 runs across 20 files**, including the complete UHD constructor and native EME mock flows on 26.15, exact script-origin enforcement, version-pair mismatch rejection, NVIDIA/AMD/Intel/unknown GPU selection, one-use startup tickets and return to the previous setting. The original Full HD tests remain included. These tests use mocks; they do not prove actual 4K playback or physical HDCP compatibility. No live 4K session was performed while preparing this build.

Disney+ [does not officially offer 4K in computer browsers](https://help.disneyplus.com/en-GB/article/disneyplus-en-lv-data-usage). This unofficial experiment does not establish official support or terms compliance. A subscription that includes 4K and capable hardware alone do not guarantee this browser path will work.
