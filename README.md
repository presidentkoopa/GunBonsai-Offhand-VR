# Gun Bonsai — Offhand / VR Support

Dual-wield support for [Gun Bonsai](https://github.com/ToxicFrog/doom-mods) 0.10.6
on VR builds of GZDoom where the player holds a weapon in each hand.

Load **after** `GunBonsai-0.10.6.pk3`.

## The problem

Gun Bonsai assumes one weapon: `player.ReadyWeapon`. On a dual-wield build the
offhand earns no XP, holds no upgrades, and its shots are credited to — and
modified by — whatever the mainhand happens to be. Offhand shots also push the
*mainhand's* weapon-type counters, so over time the mainhand starts being
offered upgrades for a weapon it isn't.

## What this does

- **Two live weapons.** Each hand gets its own `WeaponInfo`, XP, upgrade tree,
  and on-tick upgrade processing.
- **Credit follows the gun, not the hand.** Damage is attributed by recording
  what caused it at creation time and inheriting that down the causal chain —
  submunitions, fragments, explosions, minions, damage-over-time. The record
  holds a `WeaponInfo`, so a burn keeps paying the flamethrower after that hand
  switches weapons, or the gun is dropped, or it moves to the other hand.
- **Damage-over-time splits proportionally** between everything that
  contributed, rather than paying it all to whichever weapon touched the target
  last.
- **Two HUDs** — mainhand top-right, offhand top-left, as mirrored pairs.
- Assorted correctness: offhand-only no longer blanks the HUD, `I` can spend an
  offhand level, `RapidFire` no longer aborts on a null psprite, `Juggler`
  works on both hands.

Full detail, settings, and the mod-author `Claim()` contract: [`readme.txt`](readme.txt).

## How it works

The patch ships replacement copies of nine Gun Bonsai source files at their
original paths. GZDoom's later-lump-wins rule substitutes them; there is no
patching step. Everything else in Gun Bonsai is untouched.

```
ca.ancilla.bonsai/PerPlayerStats.zsc        two live infos, provenance registry, Claim API
ca.ancilla.bonsai/EventHandler.zsc          dual HUD render, hitscan bracket, offhand ShowInfo
ca.ancilla.bonsai/WeaponInfo.zsc            cross-hand rebind guard
ca.ancilla.bonsai/WeaponTypeInference.zsc   type hints from claimed shots
ca.ancilla.bonsai/HUD.zsc                   per-hand anchoring
ca.ancilla.bonsai/debug.zsc                 bonsai-debug,hands
ca.ancilla.bonsai/upgrades/Dot.zsc          proportional burn credit
ca.ancilla.bonsai/upgrades/Juggler.zsc      both hands
ca.ancilla.bonsai/upgrades/RapidFire.zsc    per-hand psprite, null guard
```

Any file shipped here that is *not* actually modified should be deleted — an
unchanged shadowed file is collision surface with other patches for no benefit.

## Known collisions

Other Gun Bonsai patches that shadow the same files will silently clobber, or be
clobbered by, this one — last load wins, with no warning:

| Patch | Shadows | Overlap |
|---|---|---|
| `Patch_levelup_settings` | `PerPlayerStats.zsc`, `WeaponInfo.zsc` | both |
| `Patch_xp_variants` | `WeaponInfo.zsc` | yes |
| `Patch_rapid_fire_redux` | `upgrades/RapidFire.zsc` | yes |

All three are small and mergeable into this patch's copies.

## Build

```powershell
.\build.ps1
```

Zips to `Gameplay_GB_VR_Offhand_Patch.0.10.6.pk3` and deploys to the rotation
folder. Edit the `$rotation` path in `build.ps1` to change where.

## Diagnostics

`bonsai-hands` in the console reports both hands' state plus the attribution
resolver's counters — how many shots were claimed outright, inherited, scored,
scored on a thin margin, or left unattributed. A climbing `thin` count means the
resolver is near-tying and guessing; a nonzero `dropped` count means the
provenance ring is undersized for the rate of fire.

## Licence

Gun Bonsai is by Rebecca "ToxicFrog" Kelly, MIT licensed. This is a derivative
work under the same terms.
