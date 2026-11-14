# SDK 26.15 Full HD compatibility patch

This unofficial build is **0.5.7.1**, based on upstream commit `b113aeb` (v0.5.7). It fixes a reproducible SDK compatibility rejection. Live Disney+ playback, 1080p decoding and long-session stability still require testing in Edge.

## Install in Edge on Windows

1. Extract the release ZIP to a permanent folder.
2. Open `edge://extensions`. Disable the old Disney+ extension and any Disney+ quality userscript, so only one copy runs.
3. Enable Developer mode, choose **Load unpacked**, and select the extracted **extension** folder containing `manifest.json`. Check that its version is **0.5.7.1**.
4. Reload the Disney+ page. Press **Alt+Shift+4**, select **Full HD (recommended)**, and allow the page to reload. If it starts OFF because of an earlier failure, explicitly select Full HD again.
5. Start a title you can normally play. Use Full HD SDR for NVIDIA/AMD; this patch does not add 4K support.
6. Check the panel's actual playing-video dimensions. The **HD badge is a request, not proof of 1080p**. Verify that the video advances with sound for at least 60 seconds; try a longer session after that.

If playback fails, choose **OFF** and reload, or disable this extension and reload. Copy the panel's diagnostics, including the new **SDK version** line, to investigate further. Do not share cookies, tokens, HAR files or license responses. No purchases or additional tools are needed to try this patch. It does not change your Windows, Edge or driver settings.

## Cause and scope

The original Japanese error says SDK settings were not applied, so the playback request was stopped. Despite its wording about a comparison mode, the same check runs in normal `fullhd` mode.

On 2026-11-14, the unauthenticated [Disney+ player configuration](https://pcs.bamgrid.com/v1/disneyplus?namespaces=hive-dmp,dmp-js-distribution) returned `26.15.0-owl` for both `dmp-js-distribution.document.playbackServiceUrl` and the legacy `hive-dmp` source. The site's public application loader uses this configuration. Upstream v0.5.7 accepts only `26.10.0-jasmine`, `26.13.3-mbaku`, and `26.14.2-nala` for this path.

An unlisted service is published without the settings wrapper. The early XHR playback guard then sees `sdkConfigReady=false` and throws before sending the POST. This matches the supplied Full HD diagnostic log: unrecognized SDK, zero playback requests and the SDK settings error. The user's actual loaded version was not in that log and could not be inspected directly, so its identity remains unconfirmed.

The missing wrapper was reproduced with the new version in the existing isolated runtime test before applying the fix. After the fix, the mocked SDK receives the expected options and the Full HD fetch/XHR paths continue. A second regression covers the early XHR failure leaving normal Full HD armed across reloads; it now saves OFF, marks the failure and gives recovery instructions.

## Static SDK review

Compared the public service bundle (`all_browser_es5`) and session bundle (`all_browser_es6`) for **26.14.2-nala** and **26.15.0-owl**. The relevant interfaces remain present with the same roles:

- UMD `playback-service` export, `PlaybackService.version` and prototype `createPlaybackSession(options)`.
- Per-session `clientParameters.configOverrides`, applied through the configuration manager to `hive-dmp`.
- `session.vod.playlistFiltering.maxHeight` and `session.vod.playbackAttributesConfig.resolution.max`.
- Engine `drmKeySystem` and `drmPlayReadyRecommendationFlow` settings and the existing session `enableDrmPreflightCheck` setting.

This extends the explicit allowlist by **one inspected release**. It does not accept arbitrary future versions, alter the UHD allowlist, change native DRM/license checks, extract keys, spoof HDCP or download video. Existing subscription, service policy, browser/CDM and output-protection requirements still apply. These findings are not an endorsement by Disney+ or a guarantee of service terms compliance.

SDK sources (inspection only; proprietary bundles are not redistributed):

- [26.15 service](https://hiveplayer-static.bamgrid.com/artifacts/hive/playback-service/26.15.0-owl/all_browser_es5/playback-service.js)
- [26.15 session](https://hiveplayer-static.bamgrid.com/artifacts/hive/playback-session/26.15.0-owl/all_browser_es6/playback-session.js)
- [26.14 service](https://hiveplayer-static.bamgrid.com/artifacts/hive/playback-service/26.14.2-nala/all_browser_es5/playback-service.js)
- [26.14 session](https://hiveplayer-static.bamgrid.com/artifacts/hive/playback-session/26.14.2-nala/all_browser_es6/playback-session.js)

SHA-256 of the inspected files:

| File | SHA-256 |
| --- | --- |
| 26.15 service | `AC6D88F31F8722821348B5706F15AC2250B1DDB3DDCC7355F1EBEE791A924D2E` |
| 26.15 session | `7233D7F4E2099351A67C070EA99808E927F5B4830191DBA8BA62CF1351AAD466` |
| 26.14 service | `3E95826F52F55D4AB52A05AED59B77B4006BEA451D8E7608585D0B6AE659DD34` |
| 26.14 session | `4E0AA71A54E6BB9CEA5CFFC1171DC629EF6503C446313D5C4701F052302DC60D` |

## Verification boundaries

Run `npm run build`, `npm test`, then `powershell -File scripts/package.ps1` to produce a checked package with Node.js 24 or later. The suite performs 29 offline runs across 20 test files, using simulated SDK/CDM/network objects. New cases cover the 26.15 service through fetch and XHR, unchanged settings ownership, Full HD SDR selection, unknown SDK rejection, UHD rejection, next-load recovery and sanitized version diagnostics.

No signed-in playback or real CDM execution was performed. Browser inspection ended because the computer-use tool could not reliably identify the active Edge URL. Public configuration can also differ across deployments; the new diagnostics expose the version observed by this extension for a follow-up test.
