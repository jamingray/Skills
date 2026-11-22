# MTG Arena Deckbuilder Skill

A reusable agent skill (`SKILL.md`) for building, optimizing, converting, and upgrading **Magic: The Gathering Arena** constructed decks directly from a player's card collection export.

Instead of eyeballing spreadsheets or guessing card legality, this skill instructs an AI assistant to parse collection files programmatically, verify live Arena banlists and metagames, audit mana bases and pitch-spell math, and output ready-to-import Arena decklists and wildcard crafting roadmaps.

## Supported Formats

| Format | Deck Size | Copy Limit | Sideboard | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **Standard** | 60+ | Max 4 | 0 (Bo1) or up to 15 (Bo3) | Paper cards only |
| **Alchemy** | 60+ | Max 4 | 0 (Bo1) or up to 15 (Bo3) | Includes Alchemy & rebalanced cards |
| **Pioneer** | 60+ | Max 4 | 0 (Bo1) or up to 15 (Bo3) | Paper cards only |
| **Historic** | 60+ | Max 4 | 0 (Bo1) or up to 15 (Bo3) | Includes digital & rebalanced cards |
| **Timeless** | 60+ | Max 4 (Max 1 Restricted) | 0 (Bo1) or up to 15 (Bo3) | Digital included; no Alchemy nerfs |
| **Standard Brawl** | 60 exact | Singleton (1 + 59) | None | Standard pool + color identity |
| **Brawl** | 100 exact | Singleton (1 + 99) | None | Digital + rebalances + color identity |
| **Competitive Brawl** | 100 exact | Singleton (1 + 99) | None | Separate banlist + color identity |

## Key Capabilities

- **Programmatic Collection Parsing:** Aggregates owned copies across all set printings (avoiding false negatives from multi-set reprints) and distinguishes unlimited standard basics from tracked `Snow-Covered` basics.
- **Live Banlist & Metagame Verification:** Checks current format banlists, Timeless Restricted lists, and Brawl "banned as commander" vs. "banned overall" rules before building.
- **Automated Deck Audits:** Verifies exact card counts, copy limits, ownership, color identity, colored mana sources, unique land names (for payoffs like `Field of the Dead`), and hypergeometric pitch-card density.
- **Format Conversions:** Ports decks between Arena formats (for example, Competitive Brawl to Brawl, or Timeless to Historic) while identifying and replacing orphaned synergies left behind by banned cards.
- **Wildcard Upgrade Roadmaps:** Filters out uncraftable phantom Cube or promo entries that appear in tracker exports and generates tiered upgrade guides with optional Excel/Sheets-ready `.csv` exports.
- **Direct MTG Arena Import Blocks:** Outputs properly formatted `Commander`, `Deck`, and `Sideboard` blocks for **Decks -> Import**.

## Repository Structure

```text
.
├── LICENSE
├── README.md
└── mtg-arena-deckbuilder/
    └── SKILL.md
```

## How to Use

### 1. Export Your MTG Arena Collection
Export your collection as a `.csv`, `.tsv`, or `.json` file using your preferred Arena tracker (such as Untapped.gg, MTGA Assistant, 17Lands, or MTGA Pro). The skill inspects column headers dynamically to locate card names, sets, rarities, colors, and owned counts.

### 2. Load the Skill Into Your AI Assistant
- **Agent Skills-compatible platforms:** Upload the `mtg-arena-deckbuilder` folder (or a `.skill` / `.zip` archive containing `mtg-arena-deckbuilder/SKILL.md`) into your assistant's Skills directory or skill manager.
- **Manual / custom instructions:** Copy the contents of `mtg-arena-deckbuilder/SKILL.md` into your project instructions or system prompt.

### 3. Example Prompts
- **Zero-Wildcard Competitive Build:**
  > "I've attached my MTG Arena collection CSV. Build the strongest Competitive Brawl deck I can play using only cards I already own."
- **Best-of-Three 60-Card Ladder Deck:**
  > "Using my attached collection export, build a Best-of-Three Pioneer deck (60 maindeck + 15 sideboard) with a budget of up to 4 Rare wildcards."
- **Format Conversion:**
  > "Convert this Competitive Brawl deck to standard Brawl. Replace all banned cards and re-tune any broken synergies using only cards in my collection."
- **Wildcard Crafting Guide:**
  > "Generate a prioritized upgrade roadmap and downloadable CSV for this deck using format-legal, craftable Arena cards I don't own yet, with recommended cuts and reasoning."

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
