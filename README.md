# PF2e Wand & Staff Casting

## Purpose and features

Wand & Staff Casting gives carried wands and staves native PF2e spellcasting entries on character sheets. Cast spells from the normal list while the module tracks charges on the source item.

## Setup

Foundry VTT 13 or 14 with PF2e 7.12.2 or newer 7.x, or PF2e 8.x. The manifest sets Foundry minimum 13 and maximum 14; it was verified with PF2e 8.5.0.

This is a free module. Install it with the [SpazzMods Installer](https://github.com/Spazzletopia-Studios/spazzmods-installer/releases/latest), or use the [public GitHub release](https://github.com/Spazzletopia-Studios/pf2e-wand-staff-casting/releases/latest). Enable **PF2e Wand & Staff Casting** in Manage Modules.

## Quick start

1. Open an owned character sheet.
2. With Hub enabled, open the SpazzMods dropdown in the sheet header and choose **Add Item Spellcasting**. Without Hub, use the original **Add Item Spellcasting** header action.
3. Choose a carried wand or staff and a normal spellcasting entry that can cast one of its spells.
4. Choose **Create Item Entry**.
5. Cast from the new Item entry in the character's normal spell list.

## Detailed use

### Wands

Wand spells use PF2e's embedded-spell casting. After the normal daily use, the Cast control offers one legal overcharge and rolls the PF2e DC 10 flat check. Success breaks the wand; failure destroys it. A later overcharge attempt after the daily attempt casts no spell and destroys the wand. PF2e's normal Repair action remains required. For PF2e wand records without item HP, the module shows the durable overcharge result; after a successful Repair, the owner can click the badge to mark it repaired. Repairing does not restore that day's use.

### Staves

The spell list comes from the source staff's linked spells. Each staff has its own charge counter, and casts spend charges from that staff. The selected casting entry controls wand casts; staff casts cannot exceed that caster's spell rank.

Only one staff can be prepared per actor at a time. Preparing a staff spends the selected prepared spell slots and adds their ranks to its charges. Staff Nexus makeshift staves have no base charges; the cantrip can still be used at zero charges. Its prepared spells determine the extra charges, which expire after 24 hours. Merging with a magical staff keeps the base charges. Retraining removes only the items linked to that exact Staff Nexus feat.

To remove an Item spellcasting entry, use its remove control. The module removes only its managed entry and spell cards for that source item. It does not edit user spell content.

### Staff Nexus

When a character has the Staff Nexus thesis, the setup picker asks for the owned Wizard book cantrip and 1st-rank spell. Cancel makes no changes. Setup creates a makeshift staff, a native Item entry, and two spell cards without changing the original spellbook. If the spellbook is not ready, retry with **Set Up Staff Nexus** in the sheet header. Level-Up Assistant can call the public setup API after character generation; it owns that one quiet call.

## Settings

No configurable module settings are registered.

## Limits and recovery

You must own the character to create entries or cast. The selected caster must have a normal spellcasting entry eligible for the source item. If an entry cannot be recreated because its casting entry is missing, recreate it from the picker. Review charge and repair badges before another cast; a destroyed wand cannot be used.

## API and development

The public API is `game.pf2eWandStaffCasting`. It includes `addItemCasting`, `removeItemCasting`, staff setup, charge adjustment, preparation, and repair helpers. `capabilities.staffNexus === 1` identifies Staff Nexus support.

Run the source checks with `npm --prefix harness test`. This does not replace installed checks on the supported Foundry/PF2e lines.

## Credits and license

Author: Spazz. MIT License.

## Get help

[SpazzMods Support](https://github.com/Spazzletopia-Studios/spazzmods-support).
