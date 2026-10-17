# CSV Wrangling

A Claude skill. Use when the user wants to inspect, clean, filter, sort, join, convert or run SQL queries on CSV, Excel or JSON data files.

## What's inside

- `csv-wrangling/SKILL.md`

## Install

### Claude app (claude.ai, desktop, mobile)

1. Download this repo as a ZIP (**Code > Download ZIP**) and unzip it.
2. Zip the `csv-wrangling` folder on its own, so the ZIP contains `csv-wrangling/SKILL.md`.
3. In Claude's settings, open the Skills section, upload that ZIP, and turn the skill on.

### Claude Code

```bash
git clone https://github.com/bogsdav/csv-wrangling.git
mkdir -p ~/.claude/skills
cp -r csv-wrangling/csv-wrangling ~/.claude/skills/
```

Restart Claude Code. The skill loads automatically when a request matches its description.

## Try it

- "Profile this CSV and tell me which columns have blanks."
- "Convert this Excel file to CSV and filter rows where status is Paid."
- "Run a SQL query that joins these two CSV files on customer_id."

## Credits

Built on [csvkit](https://github.com/wireservice/csvkit) (MIT license). csvkit itself is not included in this repo; the skill installs it when needed.

## License

MIT. See [LICENSE](LICENSE).
