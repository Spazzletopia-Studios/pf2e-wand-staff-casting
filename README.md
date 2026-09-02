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
