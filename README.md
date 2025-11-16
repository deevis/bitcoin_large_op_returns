# Bitcoin Large OP_RETURN Data Repository

A curated collection of large OP_RETURN data extracted from the Bitcoin blockchain. This repository contains data from transactions that exceed the historical 80-byte limit, made possible by Bitcoin Core v30's increase to 100,000 bytes per OP_RETURN output.

## Overview

This repository systematically collects and organizes OP_RETURN data from Bitcoin blocks, preserving the embedded content while maintaining security best practices. Each transaction's data is stored with comprehensive metadata including block information, transaction fees, mining pool attribution, and file type detection.

## Historical OP_RETURN Limits

Bitcoin's OP_RETURN size limits have evolved significantly over time:

- **March 2014 (Bitcoin Core v0.9.0)**: OP_RETURN introduced with a **40-byte** data payload limit
- **July 2015 (Bitcoin Core v0.11.0)**: Increased to **80 bytes** of data payload
- **February 2016 (Bitcoin Core v0.12.0)**: Allowed multiple data pushes with a **83-byte total script size** limit
  - This 83-byte limit accounts for script overhead:
    - 1 byte: `OP_RETURN` opcode (`0x6a`)
    - 1-2 bytes: Push opcode(s)
    - 1 byte: Length byte
    - **80 bytes**: Maximum data payload
- **October 2025 (Bitcoin Core v30.0)**: **Limit removed** - OP_RETURN outputs can now be up to the maximum transaction size (~4MB)

### What This Repository Collects

This repository scans for OP_RETURN outputs with **total script size > 83 bytes**, which means:

- **Data payload exceeds 80 bytes** (the historical limit from 2015-2025)
- **Total script size exceeds 83 bytes** (the limit from 2016-2025)

The 83-byte threshold ensures we capture all OP_RETURNs that exceeded the historical limits, regardless of whether they used single or multiple data pushes. This includes both:
- Scripts that exceeded the 80-byte data limit in a single push
- Scripts that used multiple pushes to exceed the 83-byte total script size limit

**Note**: The distinction between 80 bytes (data) and 83 bytes (script) is important:
- **80 bytes** = Historical data payload limit (what users could embed)
- **83 bytes** = Historical total script size limit (what Bitcoin Core enforced)
- Our scanner checks **> 83 bytes** (total script size) to capture all historically "large" OP_RETURNs

## Repository Structure

```
bitcoin_large_op_returns/
├── LICENSE                    # CC0 1.0 Public Domain Dedication
├── README.md                  # This file
└── op_return_data/
    ├── timeline_data.json     # Aggregated timeline data for visualization
    └── block_*/               # Individual block directories
        ├── tx_*_metadata.json # Transaction metadata (always present)
        ├── tx_*_raw.bin       # Raw binary data (when safe)
        ├── tx_*_decoded.txt   # Decoded text content (when applicable)
        └── tx_*.<ext>         # File with detected extension (when safe)
```

## Block Directory Structure

Each block directory (`block_<block_number>/`) contains files for each large OP_RETURN transaction found in that block:

### Files Per Transaction

For each OP_RETURN transaction, the following files may be created:

1. **`tx_<txid>_<vout>_metadata.json`** (Always present)
   - Complete metadata including:
     - Block number and timestamp
     - Transaction ID and vout index
     - Data size and file type detection
     - MIME type identification
     - Mining pool attribution
     - Transaction fee information (sats, fee rate, cost per byte)
     - Input/output counts
     - **Full hex-encoded raw data** (for analysis without downloading files)

2. **`tx_<txid>_<vout>_raw.bin`** (When safe)
   - Raw binary data as extracted from the blockchain
   - Not created for executable files or archives (security risk)

3. **`tx_<txid>_<vout>_decoded.txt`** (When applicable)
   - UTF-8 decoded text content for text-based OP_RETURNs
   - Only created when data is detected as readable text

4. **`tx_<txid>_<vout>.<ext>`** (When safe)
   - File saved with detected extension (e.g., `.jpg`, `.png`, `.pdf`)
   - Enables direct viewing/opening of images, documents, etc.
   - Not created for executable files or archives (security risk)

### Example Block Directory

```
block_923702/
├── tx_c51b491f5b29fb6848b11f78220a41978a26a90b870cbf37882ff7671c020970_1_metadata.json
├── tx_c51b491f5b29fb6848b11f78220a41978a26a90b870cbf37882ff7671c020970_1_raw.bin
├── tx_c51b491f5b29fb6848b11f78220a41978a26a90b870cbf37882ff7671c020970_1_decoded.txt
└── tx_c51b491f5b29fb6848b11f78220a41978a26a90b870cbf37882ff7671c020970_1.webp
```

## timeline_data.json

The `timeline_data.json` file provides an aggregated view of all OP_RETURN data suitable for timeline visualization and analysis. It contains:

- **Filtered content**: Only includes media files (images, video, audio) and interesting human-readable text
- **Metadata summary**: Block numbers, timestamps, file types, sizes, and preview text
- **Mining pool information**: Which pool mined each block
- **Fee information**: Transaction costs and cost-per-byte metrics

### timeline_data.json Structure

Each entry contains:
```json
{
  "id": "block_923702_txid_vout",
  "block": 923702,
  "timestamp": 1703123456,
  "date": "2024-01-15T12:34:56",
  "txid": "c51b491f5b29fb6848b11f78220a41978a26a90b870cbf37882ff7671c020970",
  "vout": 1,
  "size": 15234,
  "type": "webp",
  "mime": "image/webp",
  "miner": "Foundry USA",
  "fee": 50000,
  "feeRate": 12.5,
  "hasFile": true,
  "hasDecoded": false,
  "preview": null,
  "blockDir": "block_923702"
}
```

This file is regenerated whenever new blocks are scanned and can be used to create interactive timelines and dashboards.

## Security Considerations

### Excluded File Types

For security reasons, **executable files and archive files are not stored** in this repository:

- **`.exe` files** (Windows executables) - Excluded due to malware risk
- **`.elf` files** (Linux/Unix executables) - Excluded due to malware risk
- **`.zip` files** (ZIP archives) - Excluded due to potential malware content
- **`.7z` files** (7-Zip archives) - Excluded due to potential malware content
- **`.rar` files** (RAR archives) - Excluded due to potential malware content

For these file types:
- ✅ Metadata JSON files **are** created (contains hex data for analysis)
- ❌ Raw binary files (`.bin`) are **not** created
- ❌ Files with executable/archive extensions are **not** created

### Why Metadata is Always Preserved

Even for excluded file types, the `*_metadata.json` file contains:
- Complete hex-encoded raw data (`raw_data_hex` field)
- File type and size information
- All transaction metadata

This allows researchers to analyze the data programmatically without the security risk of storing executable or archive files that could contain malware.

## Data Collection

This repository is automatically maintained by scanning Bitcoin blocks for OP_RETURN outputs with **total script size > 83 bytes** (see [Historical OP_RETURN Limits](#historical-op_return-limits) above). The scanning process:

1. Identifies large OP_RETURN transactions
2. Detects file types using magic number signatures
3. Extracts metadata and transaction fee information
4. Saves files in organized block directories
5. Automatically commits and pushes changes (when configured)

## Statistics

- **Total Blocks Scanned**: Continuously updated
- **Total OP_RETURNs**: Continuously updated
- **File Types**: Images (JPG, PNG, WebP), Text, PDFs, Audio, Video, and more
- **Date Range**: From first scanned block to present

## Usage

This repository is designed to be used as a git submodule in projects that need access to Bitcoin OP_RETURN data. The data can be:

- Analyzed programmatically via metadata JSON files
- Viewed directly (images, text files, etc.)
- Used for timeline visualizations
- Studied for blockchain data embedding patterns

## License

This repository is dedicated to the public domain under [CC0 1.0](LICENSE). The blockchain data itself is public, and this collection is provided for research and analysis purposes.

## Contributing

This repository is automatically maintained. If you find issues or have suggestions, please open an issue in the parent project repository.

---

**Disclaimer**: This repository contains data extracted from the public Bitcoin blockchain. Some embedded content may be subject to copyright by its original creators. Use responsibly and in accordance with applicable laws.
