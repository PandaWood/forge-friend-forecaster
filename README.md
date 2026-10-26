# Forge Friend Forecaster

See which Valheim weapons and armour will survive the Forge of Potential before you spend an idol, and plan who in your group should press the button at each step.

Forge of Potential rolls are fixed in advance. Each item type and level step has its own roll, and the first one comes from your character's name, so everything can be worked out from the name alone. No save file is needed.

Unofficial fan tool. Not affiliated with or endorsed by Iron Gate AB. Builds on [Forge Oracle](https://github.com/papptimus/Forge-Oracle) by papptimus, which worked out the roll mechanics.

## Use it

Open `index.html` in a browser. Everything runs locally.

1. Type your character name exactly as in game, capitals included.
2. If friends use the same Forge, add their names. Each character has their own rolls, and only the one pressing the button counts, so routes name who does each step.
3. Pick an age (Wood & flint, Bronze, Iron, Silver, Black metal, Carapace, Flametal, Deep North), or All ages. Items are grouped by the idol the game asks for.
4. Items are ranked by how high they go from level 3 before anything breaks, shown as an ingot coloured like heated iron. Items that stop at 4 or lower are folded away.
5. To go after one item, type part of its name or its type (for example "spear") in the search box. Matches are ranked by what they cost to reach the "Aim for level" setting, 7 by default.
6. Open an item and pick a target level. The route lists every step, when to hand the item and idols to someone else, which copies to break on purpose, and the materials needed for the first copy and the rebuilds.

The page assumes these characters haven't used idols on those items yet, and that each new copy is brought to level 3 at the normal workbench or forge.

## How it works

These details come from the game's own code (`assembly_valheim.dll`) and item data, game version 1.0.20, October 2026.

1. The key is `hash(item name) + target level`, using Valheim's string hash.
2. The first seed for a key is `hash(character name) + key`. Later seeds are stored in the character file.
3. Unity's random generator is seeded with it, and one value between 0 and 1 is drawn. That is the roll. A second draw, modulo 99999999, becomes the next seed for that key. This happens on every attempt, so retries can be predicted too.
4. A roll of 0.65 or less succeeds. A failure destroys the item. Every idol has the same odds.
5. An attempt at the Forge of Potential uses one idol and nothing else. On a break, the game returns 35% (rounded up) of the crafting cost plus the cost of the item's current level, for each recoverable material.
6. The material cost of upgrading to level q is the per-level amount times q − 1 below level 4, and times 4 + (q − 4) / 2 from level 4 up.

Two things follow:

- On a first run almost nothing passes level 7. The first rolls of neighbouring levels come from neighbouring seeds, which Unity's generator turns into related values, so five successes in a row happens about once in 8,000.
- Two players together usually get much further, because one of them tends to succeed where the other fails.

## What is verified

- All 19 seeds stored in a real character file are reproduced from the character name alone, including a step attempted twice.
- Predictions matched real attempts on several items.
- Which idol each item needs, its materials and the English names come from the game's recipe data and localisation.

## Known limits

- Playing together on a server hasn't been tested. The roll uses the character pressing the button, so it should work, but check one cheap step first.
- If a character has already used idols on an item, the page's route for it is wrong. `forge_predict.py` reads the stored seeds from a character file instead.
- The item list covers game version 1.0.20. A patch can change the math, the odds or the recipes.

## Command line

`forge_predict.py` (from Forge Oracle) predicts the next steps for one item from a character file, using its stored seeds:

```
python forge_predict.py --item SpearBronze --level 3 --fch Thor.fch
```

## Credits

- [Forge Oracle](https://github.com/papptimus/Forge-Oracle) by papptimus: the original tool and roll mechanics.
- [ValheimSaveParser](https://github.com/Melichor/ValheimSaveParser) by Melichor: independent confirmation of the odds.
