---
name: mtg-arena-deckbuilder
description: Build, optimize, convert, and upgrade Magic the Gathering Arena constructed decks from a player's card collection export. Use when the user uploads an MTG Arena collection file (CSV, TSV, or JSON), asks to build a competitive or budget deck for Standard, Alchemy, Pioneer, Historic, Timeless, Brawl, Standard Brawl, or Competitive Brawl, wants to port a deck across Arena formats, or requests a wildcard crafting roadmap or upgrade CSV.
---

# MTG Arena Deckbuilder

Build collection-verified, metagame-aware decks and wildcard upgrade roadmaps for any constructed format on Magic: The Gathering Arena. Every decklist, legality check, mana count, and crafting guide is verified programmatically against the user's collection export and live format banlists.

## When to Use

- The user uploads an MTG Arena collection export (CSV, TSV, or JSON) and wants a deck built from owned cards.
- The user wants a competitive, ladder-ready, or budget deck for Standard, Alchemy, Pioneer, Historic, Timeless, Brawl, Standard Brawl, or Competitive Brawl.
- The user wants to convert an existing deck from one Arena format to another (for example, Competitive Brawl to Brawl, or Timeless to Historic).
- The user asks for a prioritized wildcard upgrade guide or downloadable crafting CSV based on cards they do not yet own.

## Format Quick Reference

Always confirm current rotation, banlists, and Restricted lists via web search before finalizing a deck. Use these structural rules as the baseline:

| Format | Deck Size | Copy Limit | Sideboard | Digital / Rebalanced Cards |
| :--- | :--- | :--- | :--- | :--- |
| Standard | 60+ | Max 4 | 0 (Bo1) or up to 15 (Bo3) | No (paper cards only) |
| Alchemy | 60+ | Max 4 | 0 (Bo1) or up to 15 (Bo3) | Yes (Alchemy + rebalances) |
| Pioneer | 60+ | Max 4 | 0 (Bo1) or up to 15 (Bo3) | No (paper cards only) |
| Historic | 60+ | Max 4 | 0 (Bo1) or up to 15 (Bo3) | Yes (digital + rebalances) |
| Timeless | 60+ | Max 4 (Max 1 Restricted) | 0 (Bo1) or up to 15 (Bo3) | Digital included; no Alchemy nerfs |
| Standard Brawl | 60 exact | Singleton (1 Commander + 59) | None | No (Standard pool + color identity) |
| Brawl | 100 exact | Singleton (1 Commander + 99) | None | Yes (digital + rebalances + color identity) |
| Competitive Brawl | 100 exact | Singleton (1 Commander + 99) | None | Yes (separate banlist + color identity) |

## Workflow

### Step 1: Capture Format, Mode, and Constraints

Confirm any critical details not stated by the user before building:

1. Target Arena format (and Best-of-One vs. Best-of-Three for 60-card formats).
2. Crafting constraint: zero-wildcard owned-only build, or a specific wildcard budget.
3. Deckbuilding preferences: preferred archetypes or colors, must-include cards, or strategies to avoid (for example, avoiding narrow color-hosers in open-field Bo1 queues).

If the user gives a clear format and collection file without extra restrictions, default to zero-wildcard owned-only and Bo1 (for casual/ladder) or clearly state whether a 15-card Bo3 sideboard is included.

### Step 2: Parse and Normalize the Collection Export

Never inspect large collection files by eye or guess card ownership. Always analyze the uploaded file with code (such as Python and pandas):

1. Inspect column headers dynamically to locate card name, set code, rarity, color, and owned quantity (commonly named `Count`, `Owned`, or `Quantity`).
2. Aggregate owned copies across all printings of each card by normalized card name. A card with `0` copies in one set may be owned in another printing. Cap usable copies at the format copy limit (4 for 60-card formats, 1 for singleton/Restricted).
3. Treat the five standard basic lands (`Plains`, `Island`, `Swamp`, `Mountain`, `Forest`) as unlimited on MTG Arena even if they are absent from the tracker export. Still verify `Snow-Covered` basic lands and all nonbasic lands in the file.

### Step 3: Verify Live Format Legality and Metagame

MTG Arena banlists, rotations, and rebalances change frequently. Before selecting a deck:

1. Search for the current banned list (and Restricted list for Timeless, or "banned as commander" vs. "banned overall" rules for Brawl variants) for the target format.
2. Search current competitive metagame tiers, winning decklists, and high-winrate synergies for that format.
3. Cross-check top meta archetypes against the user's aggregated collection to find the strongest archetype they can build cleanly with owned cards (or within their stated wildcard budget).

### Step 4: Build Around Synergies (and Handle Format Conversions Properly)

Assemble the deck around cohesive gameplans rather than isolated staples:

1. Respect all user priorities (must-include cards, avoided narrow hatepieces, or preferred win conditions).
2. When building Bo3 60-card decks, design a 15-card sideboard targeting the format's top matchups and verify ownership of sideboard copies combined with maindeck copies (total copies across maindeck + sideboard cannot exceed owned copies or 4).
3. When converting a deck between Arena formats (for example, Competitive Brawl to Brawl, or Timeless to Historic), do not stop at 1-for-1 replacements for banned cards. Audit the remaining list for **orphaned synergies** (tutors, recursion, or payoff cards whose enablers were banned), cut them, and pivot those slots into a cohesive secondary synergy package legal in the target format.

### Step 5: Run Automated Verification Audits

Before writing the final response, run a Python verification script against the drafted list to confirm every constraint holds:

1. **Card count:** Exact target size (60 maindeck + 0 or 15 sideboard; 60 total for Standard Brawl; 100 total for Brawl / Competitive Brawl).
2. **Copy limits:** Every card respects the format limit (max 4, max 1 for Timeless Restricted, max 1 for singleton Brawl formats, unlimited for basic lands).
3. **Ownership check:** For every non-basic card (and `Snow-Covered` basics), `deck_qty <= total_owned_qty` across all printings in the user's file.
4. **Legality and color identity:** Zero banned cards present; for Brawl formats, every card matches the commander's color identity.
5. **Mana and pitch math:** Verify colored source counts per color, total enters-tapped lands, fetchland targets, unique land names (when using payoffs like `Field of the Dead`), and pitch-card density using hypergeometric probability whenever the deck plays pitch spells (such as `Force of Will`, `Force of Negation`, `Solitude`, `Fury`, or `Subtlety`).

### Step 6: Present the Decklist and MTG Arena Import Block

Structure the response cleanly:

1. **Strategy and Synergies:** Explain why the archetype fits the user's collection and how its primary engines, interaction, and win conditions work together.
2. **Categorized Decklist:** Group cards by functional role (Commander, Ramp, Threats/Engines, Interaction, Tutors/Card Selection, Lands, Sideboard).
3. **MTG Arena Direct Import Block:** Provide a plain-text fenced code block formatted for **Decks -> Import** using exact English Arena card names and standard section headers (`Commander`, `Deck`, `Sideboard`).
4. **Piloting Notes:** Include brief, actionable mulligan, sequencing, and matchup tips (including format-specific rules such as no free mulligan in Competitive Brawl or sideboarding patterns in Bo3).

### Step 7: Generate Wildcard Upgrade Roadmaps (When Requested)

When the user asks for upgrades or a crafting guide:

1. Cross-reference every candidate upgrade against the user's aggregated collection counts to confirm exact copies missing (`needed_qty = target_qty - owned_qty`). Only include cards where `needed_qty > 0`.
2. Verify that every recommended card is both legal in the target format AND craftable on MTG Arena.
3. Group recommendations into clear priority tiers (Tier 1 Highest Impact, Tier 2 Synergy Upgrades, Tier 3 Flex / Sideboard) with a table listing Priority, Card Name, Wildcard Rarity, Copies Needed, Arena Set(s), Recommended Cut(s), Role, and Reasoning.
4. Summarize total wildcard cost by rarity (Mythic, Rare, Uncommon, Common).
5. When asked for a file export, generate a UTF-8 BOM CSV (`encoding="utf-8-sig"`) containing those columns plus `Owned Count` and `Craft Status` (defaulting to `Not Crafted`) so it opens cleanly in Excel or Google Sheets.

## Gotchas

- **Phantom Cube and uncraftable cards in exports:** Arena tracker CSVs often include `Count = 0` entries for Arena Powered Cube, Vintage Cube, or uncraftable promo cards (such as Power Nine, original dual lands, `Mox Diamond`, `Wasteland`, `Sol Ring`, or `Urza's Saga`). Always verify a card is collectible and craftable in Constructed before listing it as an upgrade.
- **Split printings causing false negatives:** A card printed in multiple sets appears on multiple rows in collection exports. Never check only the first matching row; always aggregate across all rows for that card name.
- **Standard vs. Snow-Covered basics:** Standard basic lands are often omitted from tracker exports because Arena grants unlimited copies, whereas `Snow-Covered` basics are tracked. Do not flag standard basics as missing, and do not assume unlimited `Snow-Covered` basics.
- **Alchemy rebalance mismatches:** `Historic`, `Alchemy`, `Brawl`, and `Competitive Brawl` use rebalanced digital versions of certain cards, while `Standard`, `Pioneer`, and `Timeless` use paper text (with Timeless also allowing digital-only cards). Verify card text for the target format before relying on a nerfed interaction.
- **Broken pitch or mana ratios after swaps:** Adding colorless hatepieces or utility lands can silently push colored sources or pitch-card counts below reliable thresholds. Re-run the mana and pitch-density script after every edit.
- **Orphaned synergies after ban swaps:** Simply replacing banned cards with generic staples leaves behind dead tutors or combo pieces. Always audit the full deck holistically whenever cutting core enablers.
