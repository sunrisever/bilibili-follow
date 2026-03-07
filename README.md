English | [简体中文](README_CN.md)

# Bilibili Follow Classifier

Auto-classify your Bilibili followed accounts (UP masters) into custom categories using keyword rules, and sync the results to Bilibili follow groups.

## Features

- Auto-fetch UP master info (bio, collections, video titles, tags, posting zones, etc.)
- Keyword-rule-based auto-classification + LLM secondary review
- Incremental addition of newly followed accounts
- One-click sync classification results to Bilibili follow groups
- **AI coding assistant support**: includes `CLAUDE.md` and `AGENTS.md` for Claude Code, Codex, OpenCode, and OpenClaw

## Quick Start

### 1. Install Dependencies

```bash
pip install -r requirements.txt
```

### 2. Initialize Data Directory

Copy `data_example/` to `data/` and edit the config:

```bash
cp -r data_example data
```

Edit `data/config.json` with your Bilibili cookies:

```json
{
    "bilibili": {
        "sessdata": "YOUR_SESSDATA",
        "bili_jct": "YOUR_bili_jct",
        "buvid3": "YOUR_buvid3",
        "dedeuserid": "YOUR_UID"
    }
}
```

How to get cookies: Log in to Bilibili → F12 Developer Tools → Application → Cookies → copy the fields.

### 3. Customize Classification Rules

Edit `data/classify_rules.json` to define your category system:

- **categories**: List of category names; the last one is the default
- **manual**: Manually assign specific UP masters (highest priority)
- **keyword_rules**: Keywords and weights for each category
- **zone_mapping**: Bilibili posting zone → category mapping

See `data_example/classify_rules.json` for a starter template.

### 4. Fetch Data

```bash
# Fetch all followed UP masters' detailed info
python fetch.py all
```

> Note: Bilibili API has rate limits. Posting zone data may be missing due to throttling; run `python fetch.py zones` later to backfill.

### 5. Algorithm Classification

```bash
python classify.py
```

Results saved to `data/分类结果.json` and `data/分类结果.md`. Keyword-rule accuracy is ~90%.

### 6. LLM Review (Recommended)

Feed `data/分类结果.json` and `data/up主信息汇总.txt` to an LLM (Claude, ChatGPT, DeepSeek, etc.) for secondary review. The LLM flags questionable classifications for manual correction.

### 7. Manual Review

Verify LLM-flagged items by checking UP master homepages. Fix misclassifications in `data/分类结果.json`.

Add confirmed correct classifications to the `manual` field in `classify_rules.json` to persist them.

### 8. Sync to Bilibili Groups

```bash
# Dry run first
python sync_groups.py --dry-run

# Execute
python sync_groups.py
```

The script auto-deletes old groups → creates new groups → batch-assigns UP masters.

## Daily Usage

### Add New Follows

```bash
python add_new.py <mid>
```

Auto: fetch info → classify → append to results → update summary.

### Backfill Missing Zones

```bash
python fetch.py zones
```

Re-run `python classify.py` after backfilling, as posting zones are a key classification signal.

### Other Commands

```bash
python generate_info.py                    # Regenerate info summary
python sync_groups.py --category Gaming    # Sync specific categories only
```

## Project Structure

```
├── classify.py              # Classification algorithm (keyword rules)
├── fetch.py                 # Data fetching (full/incremental/zone backfill)
├── add_new.py               # Incremental add (fetch + classify pipeline)
├── generate_info.py         # Generate human-readable summary
├── sync_groups.py           # Sync to Bilibili follow groups
├── requirements.txt
├── data_example/            # Template configs (copy to data/)
│   ├── config.json
│   └── classify_rules.json
└── data/                    # Personal data (gitignored)
    ├── config.json
    ├── classify_rules.json
    └── ...
```

## Notes

- `data/` contains personal data and is gitignored
- Running `classify.py` overwrites previous results (save manual overrides in the `manual` field)
- Group names cannot contain `/` (auto-stripped)
- First-time workflow: copy data dir → fill cookies → fetch → classify → LLM review → manual review → sync

## License

MIT
