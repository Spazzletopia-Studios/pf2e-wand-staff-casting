# PF2e Wand & Staff Casting

Standalone character-sheet support for carried wands and prepared staves on
PF2e 8.4.1. **Add Item Spellcasting** creates a real native PF2e spellcasting
entry for the selected item, so its spells appear in the normal Spellcasting
list with the usual spell cards and Cast controls.

- Wands use PF2e's native embedded-spell cast path. After the normal daily use,
  the same Cast control offers the one legal overcharge and rolls PF2e's DC 10
  flat check.
- Staves read their source spell UUID links directly from the staff item. The
  linked spell cards are created in that item's native entry and Cast spends
charges from that specific staff.
- Staff charges are stored on the individual staff under
  `flags.pf2e-wand-staff-casting.staff`. The module enforces the normal
  one-prepared-staff-per-actor rule while keeping every item counter separate.
- Each source and spell card stores its source UUID and item id. The generated
  cards are module-managed and can be removed as one entry; user spell content
  is never edited.

Rules checked: *GM Core* pages 278 and 282 through Archives of Nethys, plus
Paizo's official errata. The harness also checks the installed PF2e 8.4.1 API
and real system/world data.

This module does not require Rest Flow. When both modules are active, Rest Flow
uses the public `game.pf2eWandStaffCasting` helpers to prepare, show, and adjust
this same per-staff charge pool.

Version 0.1.1 preserves planned document ids when creating an entry and repairs
the broken spell-to-entry links made by 0.1.0. If an empty entry was deleted,
adding that item again reuses its saved spell cards instead of making copies.
Every picker option also has literal dark and light colors, so system/theme
option styling cannot make the dropdown illegible.

Version 0.1.2 adds the optional Rest Flow charge API. It also clears PF2e's
slot-expended marker from module-managed item spell rows, so spell names and
charge badges are not crossed out. Wand overcharge now posts an explicit DC 10
PF2e flat check, reports success/failure, destroys the wand on failure or a
later forbidden attempt, and locks the item while a cast is still resolving.

Version 0.1.3 queues fast repeated clicks instead of dropping the legal second
cast. A wand now has one safe cast, then one overcharge that casts before its
visible DC 10 flat check. Success makes the wand broken; failure destroys it.
Any later overcharge attempt destroys it without a spell or another check. A
restored daily use clears the prior day's overcharge state on its safe cast.

Version 0.1.4 fixes cast controls after a PF2e sheet redraw. Foundry can replace
the Cast button while copying its old HTML marker; that left the new button
without the module handler and let PF2e cast an exhausted wand as a normal
spell. Binding identity is now tracked on the real button node, so every new
button is bound once and the overcharge path survives redraws. The flat check
uses PF2e 8.4.1's public `game.pf2e.Check` and `CheckModifier` runtime API.

Version 0.1.5 adds a compact high-contrast **BROKEN** badge beside every spell
linked to a wand that survived its overcharge check. Its tooltip says that the
wand needs repair, and the badge is rebuilt from the durable wand state after
every PF2e sheet redraw.

Version 0.1.6 keeps every rank listed by a staff as its own native spell card,
including inherited spell lists such as Bounty's Light. Descriptive spell links
outside the ranked list are no longer treated as staff spells. Existing managed
entries gain their missing ranked cards on the primary GM client without
duplicating valid cards. Wand casts now honor the casting entry chosen at setup;
staff casts cannot exceed that caster's spell rank; and removing or destroying
an Item entry deletes only module-owned cards. A repaired wand that was already
overcharged shows **OVERCHARGED** until its daily use resets, while a physically
broken wand continues to show **BROKEN**. PF2e wand records that have no item HP
use the durable overcharge outcome for the same badge; after the normal Repair
action succeeds, its owner can click the badge to mark the wand repaired without
clearing that day's overcharge.

## 0.1.7 source preview: Staff Nexus

An owned Staff Nexus feat starts a compact two-spell picker on the client that
added it. It waits until a Wizard spellbook has a cantrip and a 1st-rank spell.
Cancel writes nothing. **Set Up Staff Nexus** in the sheet header is the retry
action. Setup creates a makeshift staff, a persistent native Item spellcasting
entry, and two native spell cards. These are real embedded Items, not temporary
sheet entries. The retry link updates even if the sheet was open before the
thesis was added: Foundry AppV1 retains the header on body redraws. The original
spellbook is not changed. Retraining removes only
the new Items tagged to that exact thesis feat.

Makeshift staves receive no base charges. Their cantrip works at zero charges.
Prepared spells add their ranks as charges: one spell below level 8, two from
level 8, and three from level 16. Those charges expire after 24 hours. A merged
magical staff retains its normal base charges. A flag without an owned thesis
does not enable these rules. Native entries use `proficiency.slug:null` to use
PF2e's base spellcasting rank, with INT for Staff Nexus. Wizard identity belongs
in the source entry's class flag, not in a class-DC statistic slug. Native Cast uses the selected Wizard entry and
the staff's charge pool, with no second spell-slot charge.

The public API is `game.pf2eWandStaffCasting`:

- `capabilities.staffNexus === 1` is the synchronous feature check.
- `setupStaffNexus(actor, {interactive=true, staffId=null, spellIds=null})`
  is async. Spell IDs are the owned book cantrip followed by the owned 1st-rank
  spell. Quiet generation must pass `interactive:false`; with no configured
  staff, pass explicit spell IDs. An existing valid selection is reused.
- Success returns `{ok:true, staffId, entryId, spellIds, createdIds,
  existingItemChanges}`. Each change is `{itemId,before:flatpatch}`; missing
  keys use Foundry's `-=key` removal form. IDs cover only this call's new Items.
- Cancellation returns `{ok:false,cancelled:true}`. A book not yet ready returns
  `{ok:false,deferred:true,reason:'spellbook-not-ready'}`. Invalid quiet choices
  return `{ok:false,reason}`. Write failures roll back and throw.
- `undoStaffNexus(actor,result)` restores the old staff fields and removes only
  those new IDs. Use it if the caller cannot save its undo journal.
- `staffNexusReady(actor,staff)` is a pure check returning `{ok,reasons,...}`.
  It validates the owned thesis, selected book spells, Wizard entry, and native
  spell links. It ignores Level-Up's old `runtimeBlocker` flag; setup clears that
  flag only after the check succeeds, and records the change for undo.
- `prepareStaff(actor,staffId,casterId,bonusTokens)` accepts an array of distinct
  `entryId|slotKey|slotId` tokens from available prepared slots. It returns
  `{ok,value,max,bonusRank}`. The native charge badge opens the same picker.

The watcher checks the initiating `userId` and
`game.pf2eLevelUpAssistant.isGeneratingCharacter(actor)`. Level-Up owns its one
quiet post-gear call, including when gear is off. No broad global quiet flag is
used. Older Level-Up integrations must provide the actor-scoped generation API
before enabling automatic thesis grants.

Browser selectors: `.pf2e-wsc-dialog select[name="rank0"]`, `[name="rank1"]`,
`[data-button="confirm"]` (Set up staff), `[data-button="cancel"]` (Cancel).
Retry is `a.pf2e-wsc-nexus`. Preparation uses `[data-wsc-charges]` and
`select[name="bonus0"]`, plus `bonus1`/`bonus2` at the required levels;
its confirm button reads Prepare. Selectors must be scoped to the open dialog.

Source and stand-in browser tests do not replace the parent's final live check.
No installation or deployment is included in this preview.

## Get help

[Get Help](https://github.com/Spazzletopia-Studios/spazzmods-support) — report a bug, get install help, ask a question, or suggest an idea.
