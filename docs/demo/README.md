# TTAPI Demo Videos

This folder contains demo videos showcasing TTAPI's performance.

## Videos

### 1. Initial Run (Cold Cache)
**File:** `TTAPI 1st run 2025-11-13.mov` (8.4 MB)  
**Duration:** ~3 minutes  
**Description:** Full data collection pipeline from scratch

**What's shown:**
- Core Data Collection: 22,348 symbols, 76,109 trading days
- Symbol Data Collection: 507 S&P 500 symbols
- EOD Data Processing: 29.2M rows imported
- Statistical Analysis: Volatility calculations

**Performance:**
- Total time: 3m 2s
- Import throughput: 438,000 rows/second
- Memory usage: 2.4 GB peak

### 2. Cached Run (TTL-Based Cache Hits)
**File:** `TTAPI 2nd run 2025-11-13.mov` (3.1 MB)  
**Duration:** ~8 seconds  
**Description:** Same pipeline with intelligent caching

**What's shown:**
- Instant cache hits from Parquet files
- Sub-millisecond operations (most complete in 16ms)
- Automatic TTL validation
- Same results, 23x faster

**Performance:**
- Total time: 7.8s (23x faster)
- Symbol data: 88ms (590x faster)
- EOD data: 3.978s (36x faster)
- Cache hit rate: 100%

## Viewing

These videos are embedded in the [Demo Videos page](ttapi-demo-script.md) and served directly by GitHub Pages.

**Direct links:**
- [Initial Run](TTAPI 1st run 2025-11-13.mov)
- [Cached Run](TTAPI 2nd run 2025-11-13.mov)

## Technical Details

**Format:** QuickTime (.mov)  
**Codec:** H.264  
**Resolution:** 1920x1080 (estimated)  
**Compatibility:** Modern browsers support HTML5 video playback

## Privacy & Security

✅ These videos show only:
- Public market data (symbols, prices, volumes)
- Performance metrics (timing, throughput)
- Console output (progress indicators, grid layout)

❌ No sensitive information:
- No API keys or tokens
- No OAuth credentials
- No personal account information
- No proprietary trading strategies

## Recording Details

**Recorded:** November 13, 2025  
**Environment:** sky [production]  
**Version:** v0.1.281 [RELEASE]  
**Profile:** sky (production OAuth2)

**System:**
- macOS (Darwin)
- 8-core CPU (800% utilization during parallel operations)
- 16+ GB RAM (2.4 GB peak usage)

## File Sizes

| Video | Size | Duration | Bitrate |
|-------|------|----------|---------|
| Initial Run | 8.4 MB | ~3m 2s | ~380 kbps |
| Cached Run | 3.1 MB | ~8s | ~3.2 Mbps |

## GitHub Pages

These videos are served directly from the GitHub repository via GitHub Pages. No external video hosting (YouTube, Vimeo) is required.

**Advantages:**
- ✅ No external dependencies
- ✅ Full control over content
- ✅ No ads or tracking
- ✅ Works offline (if repo is cloned)

**Limitations:**
- ⚠️ No streaming optimization (full download required)
- ⚠️ No adaptive bitrate
- ⚠️ Limited to GitHub's file size limits (100 MB per file)

## Future Updates

If you need to update the videos:

1. Record new videos with the same naming convention
2. Replace the .mov files in this directory
3. Update the file sizes in this README
4. Commit and push to GitHub

The demo page will automatically use the new videos.

