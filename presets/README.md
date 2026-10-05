# Fitting presets (`presets/`)

Static preset data that fitting tools bundle but CCP's SDE does not ship as such: damage profiles, target profiles,
NPC damage types, implant sets, character skill presets, search aliases. Generated, not hand-copied:

```bash
python presets/make_presets.py --sde SDE_JSONL_DIR --out presets   # aliases read from ../aliases/aliases.json
SDEPIPE_PRESETS=presets python -m unittest discover -s presets/tests
```

Pure stdlib and deterministic: the same inputs give byte-identical output. The generated `presets.json` is **not
committed** — it ships as a release asset of `sde-<build>-r<rev>` (`gh release download -R EX-CT/EXFA-Data
-p 'presets.json'`). The counts below are from SDE build 3569502 (2026-10-02).

`presets.json` is derived from the SDE by the rules in `make_presets.py`, plus EX-CT hand-written tables
(classification keywords, the alias list in `aliases/aliases.json`, generic profiles). Licences: the data is under
CCP's third-party developer licence (`LICENSE.EVE`); the rules and hand-written tables are LGPL-3.0-or-later
(`LICENSE`). Every section carries a `provenance` object with: source, SDE build and files, method, generator,
`license`.

The Pyfa-derived tables that used to ship as `presets-pyfa-LGPL-GPL.json` are **not** produced here any more —
Pyfa-related tooling is GPL and lives in [EX-CT/EXFA-Bench `oracle/`](https://github.com/EX-CT/EXFA-Bench/tree/main/oracle).
The last published copy remains on the legacy release
[`EX-CT/eve-sde-pipeline` `sde-3569502-r5`](https://github.com/EX-CT/eve-sde-pipeline/releases/tag/sde-3569502-r5).

## Sections of `presets.json` (SDE 3569502)

| section | count | what / how |
|---|---|---|
| `damage_profiles.generic` | 5 | Uniform, and pure EM / Thermal / Kinetic / Explosive (definition) |
| `damage_profiles.ammo` | 823 | every published charge (category 8) with damage. Base damage and ratio, no skills or launcher bonuses |
| `damage_profiles.npc` | 69 | per NPC faction (19) and per faction × context (asteroid belt, deadspace, mission, FW, incursion, …). `ratio` is weighted by DPS; `ratio_type_mean` is the mean of the per-type ratios |
| `target_profiles.generic` | 5 | ideal target, and uniform 25 / 50 / 75 / 90 % |
| `target_profiles.npc` | 106 | median over the NPC types of a (faction, hull class): resists per layer and HP-weighted, signature radius, max velocity, radius, HP |
| `npc_damage_types.factions` | 19 | damage share per faction, primary damage type, and secondary (share > 5 %) |
| `npc_damage_types.types` | 5185 | every armed NPC type: DPS by damage type, share, primary types |
| `implant_sets` | 47 | published implants carrying an `implantSet*` set-bonus attribute (multiplier; per-slot `...Modifier` and `...FAKE` display attributes excluded), grouped by attribute and Low/Mid/High grade. Members with slot (`implantness`) and multiplier; `complete` means all 6 slots are present |
| `character_skill_presets` | 10 | All 0 … All 5 (every published skill, 512) and the SDE clone grades (alpha clone caps) |
| `search_aliases` | 108 | EX-CT abbreviation table in `aliases/aliases.json` (mwd, scram, lse, bcs, haml, rf, …) resolved to type ids by whole-word match on English names. 6 aliases with no match are listed in `dropped_no_match` |

NPC rules: NPCs are SDE category 11. The faction comes from `factionID` when the type has one, otherwise from
keywords in the group name. Context comes from the group-name prefix and hull class from a word in the group name.
A weapon counts only when the type has the matching weapon effect: turret `targetAttack` / `projectileFired` /
`targetDisintegratorAttack`, or missile `missileLaunchingForEntity`. Many NPCs carry leftover weapon attributes
with no effect. Turret DPS = damage × `damageMultiplier` / `speed`. Missile DPS = the damage of the
`entityMissileTypeID` charge × `missileDamageMultiplier` / `missileLaunchDuration`.

## Gaps / known limits

- NPC profiles are not weighted by spawn frequency or site composition, because the SDE has no spawn tables. Pyfa's
  curated mission and abyssal profiles, e.g. Abyssal per weather/tier, cannot be derived from the SDE.
- Abyssal, Irregular, Homefront and similar NPC groups without a faction keyword or factionID are left out of the
  faction aggregates. They are still in `npc_damage_types.types` with `faction: null`.
- NPC DPS uses base attributes. It ignores NPC behaviour (orbit range, target switching), and it ignores fighters,
  drones and EWAR, plus anything not modelled as entity turret or missile attributes.
- The search aliases are English only and hand-curated. Pyfa's larger jargon table is GPL — see the oracle note above.
- Not included: Pyfa market overrides (force-published or regrouped items), price data, and saved user fits and
  characters. ESI was not needed: everything above comes from the SDE.
