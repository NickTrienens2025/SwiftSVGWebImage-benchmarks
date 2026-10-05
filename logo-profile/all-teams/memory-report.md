# Memory for all 32 NHL logos at 1200×900

Each team runs in its own fresh release-test process. Source data is prefetched into a memory-only cache before the baseline; there is no image/Core Image warmup. One 1200×900 render uses Lanczos and retains its CGImage until the memory readings are taken. Bitmap size is CGImage bytesPerRow × height. Encoded source and PNG storage are cache-owned bytes. Peak increase is the change in getrusage high-water RSS; RSS increase is the change in current resident size. Both include native temporaries, framework/driver initialization, and allocator effects. They are not precise allocation attribution or retained-heap measurements. One isolated trial per team; small differences may be noise. Because framework resources are shared in a real app, summing these peak increases would overestimate a concurrent workload.

| Team | Bitmap MiB | Source KiB | Cached PNG KiB | Peak increase MiB | RSS increase MiB | Total peak MiB |
| --- | --- | --- | --- | --- | --- | --- |
| Anaheim Ducks | 4.12 | 15.1 | 177.3 | 67.31 | 39.67 | 83.81 |
| Boston Bruins | 4.12 | 2.3 | 87.1 | 67.08 | 34.16 | 83.56 |
| Buffalo Sabres | 4.12 | 9.0 | 188.4 | 67.80 | 38.97 | 84.25 |
| Calgary Flames | 4.12 | 5.0 | 150.2 | 65.23 | 38.31 | 81.69 |
| Carolina Hurricanes | 4.12 | 2.8 | 165.6 | 63.36 | 38.09 | 79.86 |
| Chicago Blackhawks | 4.12 | 25.4 | 327.3 | 66.19 | 38.86 | 82.61 |
| Colorado Avalanche | 4.12 | 5.9 | 243.7 | 65.11 | 38.31 | 81.55 |
| Columbus Blue Jackets | 4.12 | 5.8 | 246.6 | 66.42 | 38.50 | 82.91 |
| Dallas Stars | 4.12 | 3.9 | 135.2 | 68.98 | 40.33 | 85.41 |
| Detroit Red Wings | 4.12 | 15.3 | 274.4 | 68.20 | 40.64 | 84.66 |
| Edmonton Oilers | 4.12 | 4.4 | 158.8 | 67.30 | 38.34 | 83.72 |
| Florida Panthers | 4.12 | 13.1 | 197.2 | 66.64 | 38.28 | 83.03 |
| Los Angeles Kings | 4.12 | 34.3 | 200.3 | 65.33 | 16.81 | 81.77 |
| Minnesota Wild | 4.12 | 4.1 | 152.1 | 64.11 | 38.19 | 80.59 |
| Montréal Canadiens | 4.12 | 2.4 | 171.6 | 64.77 | 34.25 | 81.17 |
| Nashville Predators | 4.12 | 7.6 | 193.8 | 62.70 | 38.22 | 79.19 |
| New Jersey Devils | 4.12 | 3.6 | 149.0 | 66.34 | 38.14 | 82.80 |
| New York Islanders | 4.12 | 9.7 | 192.3 | 66.05 | 34.19 | 82.48 |
| New York Rangers | 4.12 | 5.0 | 156.2 | 65.02 | 38.02 | 81.47 |
| Ottawa Senators | 4.12 | 23.6 | 264.8 | 68.88 | 40.78 | 85.41 |
| Philadelphia Flyers | 4.12 | 2.2 | 98.1 | 63.08 | 37.91 | 79.52 |
| Pittsburgh Penguins | 4.12 | 9.3 | 171.1 | 66.73 | 38.50 | 83.14 |
| San Jose Sharks | 4.12 | 9.4 | 181.8 | 68.72 | 40.41 | 85.19 |
| Seattle Kraken | 4.12 | 4.4 | 153.7 | 66.38 | 38.14 | 82.80 |
| St. Louis Blues | 4.12 | 2.6 | 129.0 | 64.02 | 16.06 | 80.45 |
| Tampa Bay Lightning | 4.12 | 1.3 | 104.4 | 66.22 | 38.00 | 82.77 |
| Toronto Maple Leafs | 4.12 | 19.7 | 168.8 | 67.97 | 38.78 | 84.45 |
| Utah Mammoth | 4.12 | 4.5 | 204.4 | 68.12 | 40.47 | 84.62 |
| Vancouver Canucks | 4.12 | 6.9 | 245.1 | 66.98 | 38.47 | 83.44 |
| Vegas Golden Knights | 4.12 | 2.1 | 121.5 | 66.64 | 37.95 | 83.11 |
| Washington Capitals | 4.12 | 7.6 | 206.1 | 66.30 | 38.61 | 82.70 |
| Winnipeg Jets | 4.12 | 3.6 | 178.9 | 67.56 | 38.38 | 84.02 |

## Whole gallery batch

The separate 32-team gallery batch finished in 707.5 ms. Peak process RSS was 148.0 MiB; end RSS 68.3 MiB; RSS after clearing encoded cache and waiting 250 ms 40.4 MiB. Encoded cache storage was 5.92 MiB. Maximum observed queue depth: 30 waiting, 2 active. This is a different workload from the isolated measurements: all sources are prefetched, render results are exported to PNG files and released, and process memory is sampled about every 2 ms. Brief peaks may fall between samples; the stated peak uses getrusage.

Reproduce with `Scripts/profile-all-team-memory.sh`.
