# FateSign BaZi MCP Server

A free remote MCP server that calculates BaZi (Four Pillars of Destiny, 八字 / 四柱推命 / 사주) charts for Claude, ChatGPT, Cursor and any other MCP client.

Language models often get BaZi charts wrong when they calculate them unaided. This server gives the model a calendar engine to call, so the pillars it interprets are calculated, not guessed.

- **Endpoint:** `https://fatesign.com/mcp` (streamable HTTP)
- **Authentication:** none. No sign-in, no API key
- **Tool:** `calculate_bazi_chart` (read-only)
- **Registry name:** [`com.fatesign/bazi`](https://registry.modelcontextprotocol.io/v0/servers?search=com.fatesign/bazi) in the official MCP Registry
- **Claude Connectors Directory:** [FateSign BaZi](https://claude.ai/directory/fatesign-bazi)
- **Glama:** [FateSign BaZi MCP connector](https://glama.ai/mcp/connectors/com.fatesign/bazi)
- **Languages:** English, Simplified Chinese, Traditional Chinese, Japanese, Korean

This repository holds the documentation and client configuration examples. The server is hosted by FateSign; its source code is not in this repository.

## Why use a tool for this

We asked 13 language models for the Four Pillars of 12 benchmark births, without tools, on 2026-11-23:

- With a plain prompt, 47% of the charts were fully correct. The best model reached 84.7%.
- The same model gave three different answers to the same question in 42% of cases.
- The hardest cases involved daylight saving time and True Solar Time.

Full results, raw answers and code: [Can AI models calculate a BaZi chart?](https://fatesign.com/en/methodology/llm-benchmark/) · [bazi-benchmark on GitHub](https://github.com/StarProtocolTeam/bazi-benchmark)

## What the engine handles

- **Time zones and historical daylight saving.** The birth time is converted with the rules in force at that place and date.
- **True Solar Time.** The chart time is corrected for the longitude of the birthplace and the equation of time.
- **Solar-term boundaries.** Year and month pillars change at the exact solar-term time, not on a calendar date. The BaZi year starts at Li Chun.
- **Late Zi hour (23:00–23:59).** The day pillar stays on the same day; the hour pillar is the Zi hour.
- **Lunar dates.** Chinese lunar-calendar input, including leap months.
- **Unknown birth time.** The hour pillar is left out instead of guessed.
- **Southern Hemisphere births.**

The calculation rules are documented at [fatesign.com/en/methodology](https://fatesign.com/en/methodology/), with [12 public test cases](https://fatesign.com/en/methodology/benchmark/).

## Connect

### Claude

FateSign BaZi is in the Claude Connectors Directory. In Claude, open **Customize → Connectors**, search for **FateSign** and choose **Connect**. No sign-in is needed. It works in Claude on the web, desktop and mobile.

### Claude Code

```bash
claude mcp add --transport http fatesign https://fatesign.com/mcp
```

### ChatGPT

Open `chatgpt.com/plugins`, choose **Add**, then **Add custom MCP server**. Enter `https://fatesign.com/mcp` and choose no authentication. Your account or workspace must allow custom MCP servers.

### Cursor

Add this to `~/.cursor/mcp.json` (see [`examples/cursor-mcp.json`](examples/cursor-mcp.json)):

```json
{
  "mcpServers": {
    "fatesign": {
      "url": "https://fatesign.com/mcp"
    }
  }
}
```

### VS Code

Add this to `.vscode/mcp.json` (see [`examples/vscode-mcp.json`](examples/vscode-mcp.json)):

```json
{
  "servers": {
    "fatesign": {
      "type": "http",
      "url": "https://fatesign.com/mcp"
    }
  }
}
```

### Any other client

Add `https://fatesign.com/mcp` as a remote server over streamable HTTP. No headers are required.

## Tool: `calculate_bazi_chart`

Calculates a BaZi chart for one person from their birth details.

### Input

| Field | Type | Required | Description |
|---|---|---|---|
| `birth_date` | string | yes | `YYYY-MM-DD`. With `calendar: "lunar"` this is the lunar year, month and day |
| `gender` | `"male"` \| `"female"` | yes | Sets the direction of the luck cycles |
| `birth_time` | string \| null | no | Local clock time at birth, 24-hour `HH:MM`. `null` when unknown. Do not adjust it for daylight saving or solar time |
| `birth_place` | string | yes, unless `timezone` is given | City and country, for example `Osaka, Japan`. Used for the time zone, daylight saving and True Solar Time |
| `timezone` | string | no | IANA time zone such as `Asia/Shanghai`. Only when the birthplace is unknown; True Solar Time is then not applied |
| `calendar` | `"solar"` \| `"lunar"` | no | Default `solar` |
| `lunar_leap_month` | boolean | no | `true` when a lunar date falls in a leap month |
| `lang` | `"en"` \| `"zh"` \| `"zh-hant"` \| `"ja"` \| `"ko"` | no | Language of the labels in the result. Default `en` |
| `focus_year` | integer | no | Year for the annual and monthly pillars, 1900–2100. Defaults to the current year |

### Output

- The four pillars, each with hidden stems, Ten Gods, Twelve Stages, Na Yin and Shen Sha
- The Day Master, the Five Elements balance and seasonal strength
- Relations between the pillars
- The luck cycles (Da Yun), the annual pillars of the current cycle and the monthly pillars of the focus year
- The birthplace that was matched and every time correction that was applied
- Warnings when the birth is close to a boundary

### Example

Request:

```json
{
  "birth_date": "1990-05-20",
  "birth_time": "14:30",
  "birth_place": "Toronto, Canada",
  "gender": "male",
  "lang": "en"
}
```

The response shows how the clock time became the chart time:

| Step | Value |
|---|---|
| Input | 14:30 |
| Time zone | America/Toronto (UTC-04:00, daylight saving time) |
| Daylight saving time | −60 min → standard time 13:30 |
| True Solar Time | 13:15 (longitude 79.38°W, −14 min) |
| Chart time | 13:15 · 未 hour |

Four pillars: 庚午 (year) · 辛巳 (month) · 乙酉 (day) · 癸未 (hour). Day Master: 乙, Yin Wood.

### Try it with curl

```bash
curl -s -X POST https://fatesign.com/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"calculate_bazi_chart","arguments":{"birth_date":"1990-05-20","birth_time":"14:30","birth_place":"Toronto, Canada","gender":"male"}}}'
```

## Example prompts

- Read my BaZi chart: born 1990-05-20 at 14:30 in Toronto, Canada, male.
- What does 2027 look like in my chart? Born 1985-01-30 at 06:10 in Seoul, male.
- Are we compatible? Born 1996-03-08 at 08:00 in Hangzhou (female) and 1994-11-20 at 22:00 in Beijing (male).

## Privacy and limits

- Birth details are used only to calculate the chart. FateSign does not store or log them, does not link them to an account and does not use them to train AI models. See the [privacy policy](https://fatesign.com/en/privacy).
- Requests are rate-limited per network address.
- BaZi is a traditional cultural framework for reflection. It is not a scientifically validated forecasting method, and the results are not medical, legal, financial or psychological advice.

## Links

- Setup guide: [fatesign.com/en/ai-assistants](https://fatesign.com/en/ai-assistants/)
- Free BaZi calculator (same engine): [fatesign.com/en/tools/bazi-calculator](https://fatesign.com/en/tools/bazi-calculator/)
- Calculation methodology: [fatesign.com/en/methodology](https://fatesign.com/en/methodology/)
- Benchmark and model test: [StarProtocolTeam/bazi-benchmark](https://github.com/StarProtocolTeam/bazi-benchmark)

## Support

Questions, errors or partnership requests: open an issue in this repository, or use the [contact page](https://fatesign.com/en/contact/).

FateSign is operated by Hyperflops G.K., Tokyo, Japan.
