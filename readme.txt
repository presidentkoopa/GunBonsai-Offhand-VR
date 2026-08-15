Gun Bonsai -- VR / Offhand Patch
================================
For Gun Bonsai 0.10.6. Load AFTER GunBonsai-0.10.6.pk3.


WHAT IT FIXES
-------------
Gun Bonsai was written for an engine where the player holds one weapon. On a
dual-wield VR build the player holds two, and stock GB has no idea. The offhand
earns no XP and gets no upgrades, and its shots are credited to -- and modified
by -- whatever the mainhand happens to be. Offhand shots also push the
MAINHAND's weapon-type counters, so over time the mainhand starts being offered
upgrades for a weapon it isn't.

This patch replaces twelve of Gun Bonsai's source files. Everything else is
stock.

  * Two live weapons. Each hand gets its own WeaponInfo, its own XP, its own
    upgrade tree, and its own on-tick upgrade processing. The offhand's
    RapidFire, Shield timer, Lightning minion and Bandoliers ammo caps now
    actually run instead of being dead or flickering.

  * Credit follows the gun, not the hand. Damage is attributed by tracking what
    actually caused it, recorded when the projectile is created and inherited by
    everything descended from it -- submunitions, fragments, explosions, minions
    and damage-over-time. Switch that hand away from the flamethrower and the
    burn still pays the flamethrower. Drop the flamethrower entirely and it
    still pays. Move it to the other hand and nothing changes, because the
    record points at the gun, never at a hand.

  * Damage-over-time splits proportionally. If two weapons burn the same enemy,
    each is paid for the share it actually contributed -- not all of it to
    whichever one touched the target last.

  * Two HUDs. Mainhand top-right, offhand top-left, as horizontal reflections of
    each other. Either side is skipped when that hand is empty, so a single
    weapon still looks like stock GB.

  * Assorted correctness. Holding only an offhand weapon no longer makes the
    entire HUD vanish. The Show Info key can now reach and spend a level the
    offhand earned. RapidFire no longer dereferences a null psprite, which with
    two hands is a VM abort rather than a glitch. Juggler works on both hands.


SETTINGS
--------
  bonsai_vr_dual_hud        Draw a HUD per hand. Default on.
  bonsai_vr_hud_hand        When dual HUD is off: 0 mainhand, 1 offhand,
                            2 whichever earned XP most recently.
  bonsai_vr_hud_main_x/y    Mainhand HUD anchor. Default 0.99 / 0.02.
  bonsai_vr_hud_main_mirror Default 3 (both axes).
  bonsai_vr_hud_off_x/y     Offhand HUD anchor. Default 0.01 / 0.02.
  bonsai_vr_hud_off_mirror  Default 2 (vertical).
  bonsai_vr_warn_binding    Startup warning when binding mode isn't per-weapon.

Set Gun Bonsai's own "upgrade binding mode" to PER WEAPON. With a weapon in each
hand, per-CLASS binding rebinds a single WeaponInfo back and forth between them,
which thrashes. The patch warns once at startup if you have not.


DIAGNOSTICS
-----------
  netevent bonsai-debug,hands      (aliased to the console command `bonsai-hands`)

Prints one line per hand -- weapon, hand flag, level, XP, upgrade count -- plus
the attribution resolver's counters:

  claimed        a mod told us outright which weapon fired (best case)
  inherited      descended from something already tracked
  scored         resolved by scoring both hands
  thin           of those, how many were near-ties, i.e. effectively guesses
  unattributed   we did not know, and awarded nothing rather than guess wrong

A high THIN count means the signals are tying and attribution is unreliable. A
nonzero DROPPED count means the provenance ring is too small for your rate of
fire. Both are meant to be watched during play; that is what they are for.


FOR MOD AUTHORS: CLAIMING YOUR OWN SHOTS
----------------------------------------
If your mod spawns a projectile and already knows which weapon fired it, say so.
That is always better than us working it out afterwards, and it costs two lines
at the spawn site where the hand is already a local variable:

    TFLV_VR_Attribution.Claim(shot, firedByWeapon);

If you convert hitscans into travelling projectiles, pass a type hint as well:

    TFLV_VR_Attribution.Claim(shot, firedByWeapon,
                              TFLV_VR_Attribution.HINT_HITSCAN);

The hint matters as much as the attribution. A converted hitscan reaches Gun
Bonsai as a bMISSILE actor moving at projectile speed, so without the hint every
shotgun and chaingun in the game quietly retrains itself as a projectile weapon
and starts drawing from the wrong upgrade pool. HINT_HITSCAN says what the
attack really was.

The hint feeds Gun Bonsai's inference counters and never touches typeflags, so a
claim can never override a weapon type you set yourself in BONSAIRC.

Mods that do not call this lose nothing; their shots fall through to scoring.


KNOWN LIMITS
------------
  * The upgrade toggle/tune menu still addresses one weapon. Gun Bonsai's netevent
    protocol has no way to name a second one; that needs a wider change than a
    patch should make.
  * Both HUDs draw the player XP bar, because that is what the frame art
    contains. Blanking one would mean shipping new textures.
  * Provenance holds live actor pointers, which do not survive a save/load or a
    level change, so it is cleared on both. A shot in flight across a transition
    goes uncredited -- deliberately better than crediting the wrong gun.


COMPATIBILITY
-------------
Uses only upstream GZDoom and QuestZDoom-lineage API. No engine-specific natives.
Requires an engine with player.OffhandWeapon and PSP_OFFHANDWEAPON.

Gun Bonsai is by Rebecca "ToxicFrog" Kelly and is MIT licensed; this patch is a
derivative of it under the same terms. See Gun Bonsai's COPYING.md.
