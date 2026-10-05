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
