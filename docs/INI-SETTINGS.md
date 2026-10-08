# Drag & Drop — INI settings

The INI is beside the loaded DLL (`SKSE/Plugins/DragAndDrop.ini`), resolved by its module path, and read once at SKSE post-load. Restart the game after changes. Shipped values below are not missing-key defaults: both contracts are preserved by the migration.

Keep values clean; put comments on separate lines. Booleans accept exact lowercase `true` or `1` as true; other strings are false. Numeric readers use `atof` and base-0 `strtol`, not strict validation. The shipped action-key line contains a trailing comment; its numeric prefix is 34. Do not copy that pattern to boolean values. Floats should be ordinary decimal numbers; integer/Skyrim.esm sound IDs may use `0x`.

## General

| Key | Shipped | Missing-key fallback | Behavior |
|---|---|---|---|
| `bEnableMod` | true | true | Enables action-key grab and drag processing |
| `bEnableLogging` | true | false | Info-level logs versus warnings/errors |
| `bShowNotifications` | false | true | Release/charge notifications |
| `iActionKey` | 34 | 34 (`0x22`) | Numeric input code, G by default; matching is device-agnostic |
| `bEnableGKeyGrab` | true | true | Action-key grab while idle |
| `bUseShoutKeyForRelease` | false | true | Exact `Shout` user-event release/charge while dragging |

## Grab

| Key | Shipped | Missing-key fallback | Behavior |
|---|---|---|---|
| `fGrabRange` | 120.0 | 150.0 | Maximum crosshair-target distance, game units |
| `fGrabHoldTimeout` | 0.5 | 0.5 | Initial hold threshold; shorter release leaves tap-grab active |
| `bGrabAnyone` | false | false | Bypasses state/follower/hostility checks, not prior player/ghost/child/weapon/immunity exclusions |
| `bGrabFollowers` | true | true | Allows player teammates |
| `bGrabChildren` | false | false | Removes child exclusion |
| `bGrabHostile` | false | false | Allows actors hostile to the player |
| `bBlockTwoHanded` | true | true | Blocks drawn two-handed swords/axes, bows and crossbows |
| `bBlockUnsheathed` | false | false | Blocks any drawn weapon |

Dead and paralyzed actors pass after the exclusions. Power-menu casts retain their pre-existing filtering limitations.

## Drag

| Key | Shipped | Missing-key fallback | Behavior |
|---|---|---|---|
| `fDragSpeedMult` | 0.5 | 3.0 | Multiplies player SpeedMult while dragging, restored on release |
| `bNoSpeedPenalty` | false | true | Suppresses engine grab-weight penalty |
| `bNoSprintWhileDragging` | false | true | Drains remaining stamina to zero while dragging |
| `fStaminaDrainRate` | 5.0 | 5.0 | Active per-second stamina drain; not a stub |
| `fDragMaxVelocity` | 5.0 | 5.0 | Per-frame ragdoll velocity cap, excluding spring body; zero disables |
| `fGrabTetherDist` | 600.0 | 600.0 | Auto-release distance from player to ragdoll center |
| `fDropOnHitChance` | 100.0 | 100.0 | Percent chance of deferred drop on a non-projectile player hit |
| `fDropOnProjectileChance` | 0.0 | 100.0 | Percent chance of deferred drop on a projectile player hit |
| `bChargeThrowOnHold` | true | false | At/after initial hold timeout, charged release instead of drop |

After tap-grab, tap the action key to drop; hold it to charge a throw. With `bChargeThrowOnHold=true`, an initial grab held past the timeout uses its total held duration for the same release/throw calculation. With false, it drops. No `bDropOnPlayerHit` setting is loaded by current code.

## Throw

| Key | Shipped | Missing-key fallback | Behavior |
|---|---|---|---|
| `fThrowImpulseMax` | 20.0 | 10.0 | Maximum throw-force ramp |
| `fThrowDropWindow` | 0.2 | 0.5 | Hold duration below this is a drop |
| `fThrowTimeToMax` | 3.0 | 4.0 | Ramp duration after the drop window; use a positive value |

Force ramps as `(heldDuration - fThrowDropWindow) / fThrowTimeToMax * fThrowImpulseMax`, capped at the maximum. The source does not validate zero/invalid ramp duration.

## Physics

| Key | Shipped | Missing-key fallback | Behavior |
|---|---|---|---|
| `fSpringDamping` | 1.5 | 1.5 | Mouse-spring damping |
| `fSpringElasticity` | 0.05 | 0.05 | Mouse-spring elasticity |
| `fSpringMaxForce` | 1000.0 | 500.0 | Mouse-spring maximum relative force |

## Sound

| Key | Shipped | Missing-key fallback | Behavior |
|---|---|---|---|
| `iGrabFailSound` | 0x4FA3B | 0 | Invalid-target feedback |
| `iGrabSound` | 0x3E633 | 0 | Grab feedback |
| `iDropSound` | 0x624AA | 0 | Drop feedback |
| `iThrowSound` | 0xC06D1 | 0 | Throw feedback |

Sound values are Skyrim.esm sound-descriptor FormIDs. Zero disables the sound.

## Impact

| Key | Shipped | Missing-key fallback | Behavior |
|---|---|---|---|
| `fImpactRadius` | 120 | 200.0 | XY proximity radius, game units |
| `fImpactDuration` | 3.0 | 3.0 | Maximum released-actor tracking duration |
| `fImpactMinVelocity` | 0.5 | 0.5 | Tracking/swing minimum Havok velocity |
| `fImpactForce` | 300.0 | 300.0 | Base mass-scaled ragdoll impulse |
| `fImpactPushForceMax` | 5.0 | 5.0 | Loaded but not forwarded; standing-actor Papyrus force remains 5.0 |
| `fImpactDamage` | 0.0 | 0.0 | Base impact damage; zero disables |
| `fImpactDamageThrownMult` | 1.0 | 1.0 | Self-damage multiplier on thrown actor |
| `bImpactOnDrop` | false | false | Released-drop tracking option |
| `fSwingImpactRadiusMult` | 0.6 | 0.5 | Multiplies impact radius during drag |
| `fSwingImpactCooldown` | 0.5 | 0.5 | Per-target swing cooldown |
| `bSwingImpactStatics` | true | true | Impulses dynamic clutter bodies during swing |
| `fRagdollMaxVelocity` | 5.0 | 20.0 | Velocity cap after impact impulse |
| `fImpactForceSpeedScale` | 1.0 | 1.0 | Existing speed-dependent force scaling |
| `fImpactDamageSpeedScale` | 1.0 | 1.0 | Existing `1 + speed * scale` damage scaling |

Source authority: `SKSE/src/DragHandler.cpp::LoadSettings`, `OnKeyDown`, `OnKeyUp`, `GetForce`, drag/impact handlers; `SKSE/Plugins/DragAndDrop.ini`; `Source/Scripts/DragDropImpactScript.psc`. These are source contracts, not proof of runtime acceptance for a new candidate.
