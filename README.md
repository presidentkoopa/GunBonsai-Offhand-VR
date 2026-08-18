# Gun Bonsai — Offhand / VR Support

Dual-wield support for [Gun Bonsai](https://github.com/ToxicFrog/doom-mods) 0.10.6
on VR builds of GZDoom where the player holds a weapon in each hand.

Load **after** `GunBonsai-0.10.6.pk3`.

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
clobbered by, this one,

## Licence

Gun Bonsai is by Rebecca "ToxicFrog" Kelly, MIT licensed. This is a derivative
work under the same terms.

vr patch conceptualized by PresidentKoopa, authored with agentic ai
