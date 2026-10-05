# EXFA-Data — EXFA · 精密装配助理 数据管线

Produces every data file the EXFA products consume: the **engine dataset** (CCP SDE → compact
`exct-eve-dataset` v1, read by [EX-CT/EXFA-Engine](https://github.com/EX-CT/EXFA-Engine)),
**fitting presets**, **search aliases** and **Jita price snapshots**.

Migrated from `EX-CT/eve-sde-pipeline` @ `83ba879` (sde/, presets/, aliases/) and
`EX-CT/eve-market-prices` @ `8d65963` (prices/). Fresh repository — no git history. The dataset format id
`exct-eve-dataset` and the snapshot format `eve-price-snapshot` v1 are unchanged; engine consumers are unaffected.

```
sde/       sdepipe/ (CCP JSONL SDE -> dataset builder + validator + differ), patches/ (+ proposed/),
           tests/, tools/make_proposed_patches.py, CHANGELOG.md
presets/   make_presets.py (damage/target profiles, NPC damage types, implant sets, skill presets,
           search aliases), tests/, README.md
aliases/   aliases.json — the search-alias table read by presets/make_presets.py
prices/    @ex-ct/exfa-prices — TypeScript package + CLI `exfa-prices`: snapshot / validate / price / rule
```

## Releases

Two kinds of GitHub Releases share this repository:

| tag pattern | workflow | marks "Latest"? | contents |
|---|---|---|---|
| `sde-<build>-r<rev>` | `sde.yml` (cron 6 h, dispatch, push to sde/presets/aliases) | **yes** | `dataset-*.json.gz`, `manifest*.json`, `CHANGELOG-<build>.md`, `diff-<build>.json`, `presets.json`, `LICENSE.EVE` |
| `prices-jita44-<time>` | `prices-snapshot.yml` (daily, dispatch) | **no** (`--latest=false`) | `prices-jita44-*.json[.gz]`, `SHA256SUMS` |

Consumers resolve the newest dataset with `gh release download -R EX-CT/EXFA-Data -p 'dataset-*'`; the price
snapshots never take the Latest slot. A successful `sde-*` release also sends a `repository_dispatch`
(`sde-release`, payload `{tag, sde_build, revision}`) to EX-CT/EXFA-Engine when the secret `EXFA_DISPATCH_TOKEN`
is configured — skipped silently otherwise.

## Commands

```bash
PYTHONPATH=sde python -m sdepipe latest                        # current TQ build number
PYTHONPATH=sde python -m sdepipe download --dest _sde          # fetch & extract the needed JSONL files
PYTHONPATH=sde python -m sdepipe build --sde _sde --out dist   # -> dist/dataset-<build>-r<rev>.json.gz + manifest.json
SDEPIPE_DIST=dist PYTHONPATH=sde python -m unittest discover -s sde/tests
python presets/make_presets.py --sde _sde --out dist-presets
SDEPIPE_PRESETS=dist-presets python -m unittest discover -s presets/tests
cd prices && npm ci && npm run build && npm test
```

`sdepipe` is pure Python stdlib; the dataset build is byte-for-byte deterministic (sorted keys, gzip mtime 0).
Dataset sections, modifier tuple layout and patches: see the `sde/` sources (unchanged from eve-sde-pipeline;
docs: [eve-fit-docs/04-sde-pipeline](https://github.com/EX-CT/eve-fit-docs/blob/main/docs/04-sde-pipeline.md)).

## Known Pyfa data drift

Known differences between Pyfa's bundled `eve.db` and the SDE live in
[EX-CT/EXFA-Bench `oracle/`](https://github.com/EX-CT/EXFA-Bench/tree/main/oracle)
(`data/pyfa-data-drift.{md,json}`); the Pyfa-side tools were moved there (GPL).

## License

Code (sde/, presets/, aliases/, prices/) — LGPL-3.0-or-later (`LICENSE`; `LICENSE.GPL-3.0` for reference).
Generated EVE data © CCP hf., CCP third-party developer licence (`LICENSE.EVE`).
