# NHL logo performance profile

2026-09-26T19:12:30Z

- Two render slots improve this 18-job Lanczos workload by 1.82× versus one slot.
- The representative memory PNG hit is 15.0× faster than generating a new PNG, writing it to disk, and decoding it.
- Lanczos increases representative cold-render latency by 1.92×. It is a visible-quality tradeoff, not a throughput optimization.
- Sustained benchmark peak RSS reaches 309.4 MiB with Lanczos versus 30.3 MiB with direct rendering. This is the main profiling concern; consult the staged values rather than treating it as per-image memory.

## Environment and methodology

```text
ProductName:		macOS
ProductVersion:		26.3.1
BuildVersion:		25D2128
Apple M3 Pro
19327352832
Apple Swift version 6.3 (swiftlang-6.3.0.123.5 clang-2100.0.123.102)
Target: arm64-apple-macosx26.0
```

Release (-O) Swift 6 build on this Mac; official NHL light SVGs loaded from local files. Six logos, three natural-aspect widths (32/64/144 points), 3× display scale: 18 variants. Each reported phase has 10 timed samples per variant; parsing has 15. Phase summary values are the median of per-variant medians, not global percentiles. One small SVG primes the renderer before timings. Each mode runs in a fresh test process. All source prefetching occurs outside timers using an in-memory transport: HTTP and network latency are excluded. Cold render includes parsing, drawing, optional Lanczos, PNG encoding, disk persistence, and PNG decoding. Memory/disk hits still decode PNG; disk-hit tests evict library memory but do not flush the macOS filesystem cache. Ten full 18-job batches use real pipeline queues and memory-only caches with PNG variants removed before each pass. Parse-once comparison alternates order across ten three-size batches, with no disk I/O. These are wall-clock phase benchmarks, not sampled per-symbol CPU attribution or device energy measurements.

## Phases

| Phase | Direct ms | Lanczos ms |
| --- | --- | --- |
| Parse SVG | 1.057 | 1.079 |
| Draw + encode (already parsed) | 0.689 | 2.008 |
| Cold PNG generation + disk write + decode | 3.040 | 5.837 |
| Memory PNG hit + decode | 0.313 | 0.390 |
| Disk PNG hit + decode | 0.499 | 0.617 |

## Queue throughput

| Mode | Jobs | Batch median ms | Jobs/sec | Peak RSS MiB |
| --- | --- | --- | --- | --- |
| direct-q1 | 18 | 58.112 | 309.7 | 29.2 |
| direct-q2 | 18 | 35.073 | 513.2 | 30.3 |
| lanczos-q1 | 18 | 94.800 | 189.9 | 310.5 |
| lanczos-q2 | 18 | 51.987 | 346.2 | 309.4 |

## Per-logo results

| Team | Width pt | Pixels | Parse ms | Direct cold ms | Lanczos cold ms | Memory ms | Disk ms | PNG KiB |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| COL | 32 | 96×64 | 1.432 | 2.611 | 5.142 | 0.302 | 0.581 | 6.7 |
| COL | 64 | 192×128 | 1.432 | 3.302 | 6.860 | 0.511 | 0.770 | 19.3 |
| COL | 144 | 432×288 | 1.432 | 5.852 | 10.562 | 1.206 | 1.564 | 62.9 |
| BOS | 32 | 96×64 | 0.550 | 1.508 | 2.952 | 0.225 | 0.438 | 4.8 |
| BOS | 64 | 192×128 | 0.550 | 1.943 | 3.680 | 0.363 | 0.591 | 12.1 |
| BOS | 144 | 432×288 | 0.550 | 3.649 | 6.590 | 0.723 | 0.924 | 27.2 |
| TOR | 32 | 96×64 | 4.771 | 6.606 | 7.629 | 0.255 | 0.483 | 6.7 |
| TOR | 64 | 192×128 | 4.771 | 6.812 | 8.725 | 0.390 | 0.643 | 18.3 |
| TOR | 144 | 432×288 | 4.771 | 8.872 | 12.682 | 0.948 | 1.315 | 51.0 |
| NYR | 32 | 96×64 | 1.194 | 2.313 | 3.775 | 0.239 | 0.456 | 5.5 |
| NYR | 64 | 192×128 | 1.194 | 2.778 | 4.376 | 0.471 | 0.701 | 14.0 |
| NYR | 144 | 432×288 | 1.194 | 4.422 | 7.299 | 0.764 | 1.050 | 43.2 |
| SEA | 32 | 96×64 | 0.963 | 2.117 | 3.173 | 0.235 | 0.438 | 6.3 |
| SEA | 64 | 192×128 | 0.963 | 2.627 | 3.873 | 0.389 | 0.581 | 17.3 |
| SEA | 144 | 432×288 | 0.963 | 4.761 | 7.712 | 1.005 | 1.195 | 47.2 |
| VGK | 32 | 96×64 | 0.483 | 1.405 | 2.623 | 0.231 | 0.431 | 4.8 |
| VGK | 64 | 192×128 | 0.483 | 1.797 | 3.216 | 0.346 | 0.537 | 12.1 |
| VGK | 144 | 432×288 | 0.483 | 3.938 | 6.532 | 0.795 | 1.021 | 33.6 |

## Reuse parsed SVG across three sizes (Lanczos)

| Team | Independent ms | Parse once ms | Speedup |
| --- | --- | --- | --- |
| COL | 13.399 | 10.190 | 1.31× |
| BOS | 9.007 | 7.704 | 1.17× |
| TOR | 25.308 | 15.296 | 1.65× |
| NYR | 12.011 | 8.827 | 1.36× |
| SEA | 11.366 | 8.788 | 1.29× |
| VGK | 9.251 | 6.921 | 1.34× |

## Memory

| Mode | Baseline | Warmup | Individual tests | Multi-size batches | Final peak |
| --- | --- | --- | --- | --- | --- |
| direct-q1 | 14.7 | 18.1 | 24.2 | 28.8 | 29.2 |
| direct-q2 | 14.7 | 18.0 | 24.0 | 28.6 | 30.3 |
| lanczos-q1 | 14.6 | 26.4 | 39.1 | 310.5 | 310.5 |
| lanczos-q2 | 14.8 | 26.4 | 39.5 | 309.4 | 309.4 |

All memory values are MiB. RSS uses getrusage(RUSAGE_SELF) in the test process (Darwin bytes), not the swift build driver. Values are lifetime high-water marks, not live heap, physical footprint, retained cache size, per-operation allocation, or a leak diagnosis. Later rows inherit earlier peaks. The staged measurements locate when the high-water mark rises. In these runs the largest Lanczos rise occurs during tight repeated low-level render/encode batches, after the individual pipeline/cache measurements. This differs from UI-paced work; the later queue benchmark inherits the existing peak. Core Image/driver resources, native temporary objects, allocator retention, and test-runner/framework overhead are included. A follow-up Allocations/VM Tracker trace and scoped autorelease-pool experiment are warranted before attributing the extra memory to a particular owner. Do not extrapolate these numbers directly to iPhone, Apple Watch, or your app.

## Recommendations

- Keep the current two-render default: the measured batch throughput benefit is substantial for both paths.
- Keep visible work ahead of speculative work. FIFO currently provides no promotion; avoid flooding it with all possible logo sizes.
- Add an optional parse-once multi-size batch for a small declared size set. The measured benefit varies by SVG; it does not remove drawing/encoding costs and should release the document at batch completion.
- Investigate sustained Lanczos memory growth before broad speculative pre-rendering. Compare Allocations/VM Tracker results with a bounded autorelease pool around synchronous native rendering; do not assume a leak or change quality defaults from RSS alone.
- Repeat on target iPhones with scrolling, memory pressure, and foreground/background priorities before setting production memory budgets. The requested Lanczos default remains unchanged by this profiling task.

## Independent reverse-order verification run

| Mode | Batch median ms | Peak RSS MiB |
| --- | --- | --- |
| direct-q1 | 57.853 | 29.1 |
| direct-q2 | 32.229 | 30.4 |
| lanczos-q1 | 93.002 | 325.7 |
| lanczos-q2 | 45.665 | 294.3 |

## Assets and reproducibility

Official assets: https://assets.nhle.com/logos/nhl/svg/{TEAM}_light.svg. NHL/team artwork remains the property of its respective owners and is not covered by the library MIT license. Images stay in ignored local build output. No fixture redistribution or release is performed.

Run `Scripts/profile-logos.sh` to rebuild and regenerate the report. Raw per-sample JSON, logs, PNGs, asset hashes, and CSV accompany the HTML report.

| Team | SVG bytes | SHA-256 |
| --- | --- | --- |
| COL | 6025 | cabc6c42f3f33a7810c0786dffcccb352c6814d83e483f745a7b07e2fe497550 |
| BOS | 2370 | 80c05e3678d3a8d2b56d46d320a2877963935677c55ec9618e506cd20e15c7e4 |
| TOR | 20146 | e8b95e832feac68796de8f3535bae6584663829a71d3f9554371cde49afb0d52 |
| NYR | 5123 | 365fc8449f0fe659eeb330c5f219416b1d3fced3ce407b359a1d403d5752ec38 |
| SEA | 4486 | 941ee0182c890a3bc0b8c4b71fe9ce98ef291d008280bfe4ed77b7715d6325d6 |
| VGK | 2194 | b02d15ccfa2510e7696a4d0533fc3f676c6e53ca2c7745e1a1a67fcbc4ff93cf |

## Larger Colorado background assets

2026-09-26T19:16:11Z

- The jersey texture is a 96×96 PNG (790 bytes), not a large SVG. It remains 96×96 for large targets; the pipeline never invents extra source resolution.
- The wire logo is a 648×431 SVG. Parsing and rendering succeed at all five tested sizes, with native transparency retained.
- At 390 points / 3×, Lanczos wire rendering takes 58.28 ms cold versus 5.23 ms for a PNG memory hit (11.1× faster).
- At 1024 points / 3×, the 3072×2044 wire output exceeds the 2× intermediate budget, so Lanczos mode correctly falls back to direct rendering.

| Asset | Width pt | Scale | Requested pixels | Decoded pixels | SVG supersample | Direct cold ms | Lanczos cold ms | Memory ms | Disk ms |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| COL_texture | 16 | 3× | 48×48 | 48×48 | — | 0.625 | 1.442 | 1.459 | 1.704 |
| COL_texture | 160 | 3× | 480×480 | 96×96 | — | 0.545 | 0.557 | 0.531 | 0.759 |
| COL_texture | 390 | 3× | 1170×1170 | 96×96 | — | 0.552 | 0.566 | 0.544 | 0.797 |
| COL_texture | 768 | 2× | 1536×1536 | 96×96 | — | 0.569 | 0.556 | 0.533 | 0.778 |
| COL_texture | 1024 | 3× | 3072×3072 | 96×96 | — | 0.538 | 0.572 | 0.539 | 0.795 |
| COL_wire | 16 | 3× | 48×32 | 48×32 | 2× | 2.598 | 3.796 | 0.172 | 0.491 |
| COL_wire | 160 | 3× | 480×320 | 480×320 | 2× | 7.493 | 15.756 | 1.189 | 1.598 |
| COL_wire | 390 | 3× | 1170×779 | 1170×779 | 2× | 28.492 | 58.283 | 5.228 | 5.973 |
| COL_wire | 768 | 2× | 1536×1022 | 1536×1022 | 2× | 47.298 | 97.562 | 8.224 | 8.837 |
| COL_wire | 1024 | 3× | 3072×2044 | 3072×2044 | 1× | 145.606 | 146.008 | 28.596 | 32.114 |

### Queue and memory

| Mode | Jobs | Batch ms | Baseline MiB | After sizes MiB | Final peak MiB |
| --- | --- | --- | --- | --- | --- |
| direct-q1 | 4 | 73.76 | 14.9 | 119.4 | 119.4 |
| direct-q2 | 4 | 46.64 | 14.8 | 130.8 | 130.8 |
| lanczos-q1 | 4 | 151.54 | 14.8 | 170.9 | 170.9 |
| lanczos-q2 | 4 | 92.72 | 14.8 | 170.6 | 179.9 |

RSS values are process high-water marks and include framework/driver resources, temporary objects, and caches. Queue measurements occur after all size tests, so their final peak can inherit memory from the 1024-point stress case. They are not per-job allocations, retained heap, or proof of a leak. The SVG supersampling cap bounds an individual intermediate surface, not all Core Image/encoding/process memory. Native 96×96 PNG decoding does not exercise Lanczos reduction at large targets; the 16-point target tests an actual 48×48 downsample.

### Methodology

Release-build measurements on the same Mac as the logo report. Four fresh processes (direct/Lanczos × one/two render slots), five timed samples after one warmup at each target. Width/scale pairs: 16pt@3×, 160pt@3×, 390pt@3×, 768pt@2×, 1024pt@3×. Natural aspect ratio is preserved: the texture targets are square; the wire targets follow 648:431. Cold timing starts from prefetched source data with no derived PNG. Memory timing renders the same snapshot again. Disk timing first reads the source with cacheOnly, then renders; macOS filesystem cache is not flushed. Thus disk timings include more work than the earlier logo-only disk-PNG benchmark. PNG inputs are decoded again on every render; they do not get persistent size-specific derivatives. Queue batches contain four jobs: texture and wire at 390pt@3× and 768pt@2×; larger stress cases are measured individually. Network/download time and preview-export encoding are excluded.

The background preview repeats the original texture as 96-pixel tiles and overlays a generated wire PNG. This is an illustrative composition of the supplied assets, not new library tiling or aspect-fill support. LoadedImage currently fits a raster image; callers must explicitly choose tiling/composition for a repeating jersey texture. Stretching this tiny PNG across a large background would enlarge the displayed pixels even though the decoder correctly avoids upscaling.

Reproduce with `Scripts/profile-backgrounds.sh`.

- [COL_texture](https://assets.nhle.com/textures/nhl/jersey/png/COL.png): 790 bytes; SHA-256 `586bb64fbd19c6b1341a18677a15441d3b79aec9afc82f2c843329933c35908e`.
- [COL_wire](https://assets.nhle.com/logos/nhl/wires/COL.svg): 7479 bytes; SHA-256 `50eeef32647cdd75af192d8ef51b7f30f68a05e67f4c45cb794b8dbc8e780ca7`.

## Isolated memory per asset and size

Per-asset memory is now measured independently rather than inferred from the earlier full-workload peak. Sizes below are point widths at the listed display scale. The standard logos and wire retain their natural aspect ratios; the texture remains square.

- 390pt @3× wire: 3.48 MiB decoded bitmap; 72.20 MiB observed peak increase during its first Lanczos render.
- 1024pt @3× wire: 23.95 MiB decoded bitmap; 73.45 MiB observed peak increase. Supersampling is skipped at this size.
- Large texture targets still return a 96×96 bitmap: 36.0 KiB decoded, plus 790 bytes of encoded cache.

| Asset | Width pt | Scale | Decoded pixels | Bitmap MiB | Direct peak +MiB | Lanczos peak +MiB | Lanczos RSS +MiB | Encoded cache KiB | Lanczos total peak MiB |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| COL_light | 32 | 3× | 96×64 | 0.023 | 3.09 | 10.97 | 10.86 | 12.6 | 27.33 |
| COL_light | 64 | 3× | 192×128 | 0.094 | 3.28 | 11.80 | 11.41 | 25.1 | 28.11 |
| COL_light | 144 | 3× | 432×288 | 0.475 | 4.23 | 17.12 | 13.73 | 68.8 | 33.53 |
| BOS_light | 32 | 3× | 96×64 | 0.023 | 3.02 | 10.78 | 10.67 | 7.1 | 27.22 |
| BOS_light | 64 | 3× | 192×128 | 0.094 | 3.11 | 11.67 | 11.28 | 14.4 | 28.03 |
| BOS_light | 144 | 3× | 432×288 | 0.475 | 3.98 | 17.34 | 14.19 | 29.5 | 33.72 |
| TOR_light | 32 | 3× | 96×64 | 0.023 | 3.25 | 11.00 | 10.86 | 26.3 | 27.44 |
| TOR_light | 64 | 3× | 192×128 | 0.094 | 3.45 | 11.88 | 11.48 | 37.9 | 28.31 |
| TOR_light | 144 | 3× | 432×288 | 0.475 | 4.28 | 17.69 | 14.47 | 70.6 | 34.11 |
| NYR_light | 32 | 3× | 96×64 | 0.023 | 3.03 | 10.84 | 10.73 | 10.5 | 27.27 |
| NYR_light | 64 | 3× | 192×128 | 0.094 | 3.23 | 11.86 | 10.98 | 19.0 | 28.27 |
| NYR_light | 144 | 3× | 432×288 | 0.475 | 4.06 | 17.22 | 14.31 | 48.2 | 33.67 |
| SEA_light | 32 | 3× | 96×64 | 0.023 | 2.92 | 10.88 | 10.77 | 10.7 | 27.22 |
| SEA_light | 64 | 3× | 192×128 | 0.094 | 3.14 | 11.75 | 11.36 | 21.7 | 28.08 |
| SEA_light | 144 | 3× | 432×288 | 0.475 | 4.03 | 17.22 | 14.14 | 51.5 | 33.61 |
| VGK_light | 32 | 3× | 96×64 | 0.023 | 2.94 | 10.88 | 10.73 | 7.0 | 27.23 |
| VGK_light | 64 | 3× | 192×128 | 0.094 | 3.05 | 11.64 | 11.25 | 14.2 | 28.00 |
| VGK_light | 144 | 3× | 432×288 | 0.475 | 4.02 | 17.34 | 13.70 | 35.8 | 33.80 |
| COL_texture | 16 | 3× | 48×48 | 0.009 | 1.94 | 10.02 | 9.97 | 0.8 | 26.39 |
| COL_texture | 160 | 3× | 96×96 | 0.035 | 2.03 | 2.00 | 1.97 | 0.8 | 18.44 |
| COL_texture | 390 | 3× | 96×96 | 0.035 | 1.95 | 1.97 | 1.94 | 0.8 | 18.39 |
| COL_texture | 768 | 2× | 96×96 | 0.035 | 2.03 | 1.95 | 1.92 | 0.8 | 18.31 |
| COL_texture | 1024 | 3× | 96×96 | 0.035 | 1.95 | 1.95 | 1.92 | 0.8 | 18.42 |
| COL_wire | 16 | 3× | 48×32 | 0.006 | 2.84 | 10.48 | 10.45 | 9.0 | 26.94 |
| COL_wire | 160 | 3× | 480×320 | 0.586 | 4.31 | 21.12 | 16.88 | 78.0 | 37.52 |
| COL_wire | 390 | 3× | 1170×779 | 3.477 | 13.81 | 72.20 | 48.38 | 275.5 | 88.55 |
| COL_wire | 768 | 2× | 1536×1022 | 5.988 | 20.70 | 116.77 | 69.55 | 409.0 | 133.09 |
| COL_wire | 1024 | 3× | 3072×2044 | 23.953 | 73.47 | 73.45 | 56.28 | 692.7 | 89.86 |

### Memory definitions

**Bitmap MiB:** Actual returned CGImage bytesPerRow × height, including row padding. This is the decoded output buffer footprint, not all process memory.

**Direct / Lanczos peak +MiB:** Increase in getrusage process peak RSS from just before to just after one render. Includes temporary rendering/encoding work and framework resources; a high-water-mark measurement, not allocation attribution.

**Lanczos RSS +MiB:** Current resident size after rendering minus baseline, using mach_task_basic_info. Output remains alive. Temporary resources may still be resident; this is not a retained-heap or leak measurement.

**Encoded cache KiB:** Original payload plus cached SVG PNG bytes after rendering. Excludes metadata, decoded images, and Core Image resources. PNG input has no derived PNG cache.

**Lanczos total peak MiB:** Whole-process high-water mark at the end of this isolated render, including the test runner and baseline.

Each of the 56 asset/size/quality cases runs alone in a fresh release-test process. The original bytes are prefetched into a memory-only cache before taking a baseline. One render then generates/decodes the requested image, with the returned CGImage kept alive while measuring memory. There is no disk cache, previous image render, or Core Image warmup. Measurements are snapshots/high-water marks, not a sum of sampled allocations. The first Core Image render includes shared initialization costs; multiple images in a real app share some of those resources, so these rows must not be added together to predict app memory. Small differences are subject to allocator and process-launch noise. The report uses one isolated trial per case; it is not a statistical memory bound.

Reproduce with `Scripts/profile-memory.sh`.

## Long queue: three independent runs

- All three independent runs completed 60 jobs successfully; observed maximum concurrency was two and maximum waiting depth was 58.
- Median completion time: 397.85 ms; range 380.36–504.76 ms.
- Whole-process peak RSS: 106.6–124.9 MiB. It is a high-water mark, not decoded-image or retained-cache size.
- After clearing encoded memory and waiting 250 ms, resident size was 57.6–59.6 MiB. Framework, allocator, source snapshots, and test-runner resources can remain; this does not establish a leak.

| Run | Jobs | Elapsed ms | Jobs/sec | Max active | Max waiting | Baseline MiB | Peak RSS MiB | End RSS MiB | After clear MiB | Encoded cache MiB |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 60 | 504.76 | 118.9 | 2 | 58 | 16.67 | 124.86 | 69.16 | 59.56 | 2.89 |
| 2 | 60 | 397.85 | 150.8 | 2 | 58 | 16.61 | 121.05 | 78.17 | 57.58 | 2.89 |
| 3 | 60 | 380.36 | 157.7 | 2 | 58 | 16.55 | 106.61 | 78.88 | 58.28 | 2.89 |

### Workload

Three sequential fresh release-test processes, using the existing Apple M3 Pro environment. Each process prefetches eight encoded sources outside the timed interval: six standard NHL logos, the Colorado wire SVG, and the jersey PNG. Every SVG is requested at eight widths (32/48/64/96/128/160/256/390 points), at 3× scale and natural aspect ratio: 56 unique SVG variants. The texture adds four requests at 16/32/64/128 points and 3×: 60 distinct source/target jobs in total. All jobs are submitted together to the default two-slot FIFO render queue with Lanczos enabled. Identical jobs are not repeated, so request coalescing cannot turn the backlog into a single render. Source data is ready, derived PNGs start empty, and the cache is memory-only with its default 32 MiB limit. Texture output never exceeds native dimensions. Returned decoded images are discarded by preRender; this models cache preparation, not 60 simultaneously retained visible images. Source snapshots remain alive throughout the measurement. No network traffic, source prefetch, or post-run 250 ms settling delay is included in completion time.

### Memory interpretation

Resident size and queue occupancy are sampled approximately every 2 ms from a separate task. Scheduling can delay samples and miss brief peaks; the peak-RSS table uses getrusage independently. The two-active limit is also checked in every captured queue sample, and the test verifies the final queue is empty and all jobs complete. Completion times include queue wait; they are not individual CPU times. Three runs show process-launch and scheduling variation, not a statistical guarantee or a leak trend. Fresh processes do not flush operating-system or GPU-driver caches. The earlier ~309 MiB tight low-level multi-size benchmark is a different workload and lifetime pattern: this experiment measures the public queued pipeline with discarded decoded results, not that tight rendering loop. No rendering or memory-management implementation was changed for this experiment.

The two-operation cap successfully bounds active work even with a 58-job backlog. For this measured workload, retaining encoded outputs is much smaller than whole-process peak RSS. Keep the current concurrency cap; prioritize visible work before filling this queue speculatively. A leak/retention investigation would require repeated batches inside one long-lived process plus allocation tracing, rather than comparing these three independent process peaks.

Reproduce with `Scripts/profile-long-queue.sh` (Python with matplotlib is required for charts; select it with `PROFILE_PYTHON`).
