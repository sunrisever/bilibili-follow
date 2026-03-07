> This file is for AI coding assistants (Claude Code, Codex, OpenCode, OpenClaw, etc.). It is optional and can be safely deleted.

# bilibili-follow-classifier

Bilibili following auto-classifier. Fetches followed UP master info via bilibili-api, classifies by keyword rules + weight scoring, syncs results to Bilibili follow groups.

## Key Commands

```bash
python fetch.py all              # Fetch all followed UP masters' info
python fetch.py zones            # Backfill missing posting zone data
python classify.py               # Run keyword-rule classification
python generate_info.py          # Generate human-readable summary
python sync_groups.py --dry-run  # Preview sync operations
python sync_groups.py            # Sync to Bilibili follow groups (destructive)
python add_new.py <mid>          # Incrementally add a new UP master
```

## Architecture

- `fetch.py`: Async data collection via bilibili-api with rate limiting (0.3s/request)
- `classify.py`: Keyword-rule matching with weighted scoring across name, bio, collections, video titles, tags, posting zones
- `sync_groups.py`: Deletes all non-default groups and rebuilds from classification results
- `add_new.py`: Pipeline for incremental additions (fetch → classify → append)
- `generate_info.py`: Generates plain-text summary for manual review
- `data_example/`: Config and rule templates
- `data/`: Personal data directory (gitignored)

## Classification Rule Format

`data/classify_rules.json` structure:
- `categories`: Array of category names (last = default)
- `manual`: `{UP_name: category}` manual overrides (highest priority)
- `keyword_rules`: `{category: {keyword: weight}}` for auto-matching
- `zone_mapping`: `{bilibili_zone: category}` for posting zone signals

## Important Notes

- `sync_groups.py` is **destructive** — it deletes all existing follow groups. Always `--dry-run` first.
- Bilibili cookies expire periodically. Update `data/config.json` when API returns errors.
- Rate limiting: 0.3s between requests, 2s pause every 50 requests.
- Group names cannot contain `/` (auto-stripped by the script).
