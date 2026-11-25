# ClinX open data server

A small program that lets an AI assistant look things up in ClinX's open data. ClinX is a platform
of tools for clinics, and its open data comes in two groups.

- **The federal 340B drug pricing program:** the state laws on the books, the bills in Congress
  and the state legislatures, the rules and notices federal agencies publish, the reports that
  hospitals, health centers and states have filed about 340B, and a calendar of their dates.
- **Health centers:** what each health center reported to HRSA in the Uniform Data System, HRSA's
  funding opportunities with their dates, and HRSA's shortage designations and health center
  sites with what changed in the last 365 days.

**Status: running at `https://clinx.org/mcp` since 15 November 2026.** Give that address to an
assistant that accepts connectors (MCP servers); there is no key and no account. It is not
published as a package yet, so to run your own copy you start it yourself, as shown below.

## What it is, in plain words

Assistants such as Claude can be given "tools": small lookups they may call while answering a
question. The Model Context Protocol (MCP) is the common way to offer such tools. This is an MCP
server with seventeen of them.

Ask an assistant "which states passed a contract pharmacy law this year?" and, with this server
connected, it does not have to guess or search the web. It asks the server, gets rows from the
same tables anyone can download from [clinx.org/data](https://clinx.org/data/), and answers with
the link to each official page.

The server holds no data of its own. It reads public files from clinx.org, keeps them in memory
for an hour, and answers from that copy. The six 340B tables are read when the first question
arrives. A health center file is read only when a health center question is asked. It only reads. It has no sign-in, no key, no
account, and it keeps nothing about who asked what.

## The seventeen tools

Eleven answer about 340B, five about health centers, and `about_the_data` covers both groups.

| Tool | What it answers |
| --- | --- |
| `find_laws` | State laws on the books that name 340B, by state, kind or words. |
| `get_law` | One law in full: citation, who it binds, what it limits, how it is enforced, dates, the passage that names 340B. |
| `find_bills` | Bills that name 340B, by state (or `US` for Congress), stage, role, kind or words. |
| `get_bill` | One bill in full, with its sponsor, stage, latest step and every dated step. |
| `find_notices` | Federal rules, proposed rules and notices that name 340B, by kind of document, role, words, day of publication, or whether comments are still open. |
| `get_notice` | One rule or notice in full: the agency that wrote it, what it does about 340B, its comment deadline, the day it takes effect and every other day it sets. |
| `calendar` | Dated steps, hearings, session days and federal dates. With no dates given: what is coming up from today. |
| `reporting_states` | The states whose law requires 340B reports, grouped by what the public can see, with what is public, where it stands, whether ClinX holds the figures, and the official sources. Filter by state or group. |
| `find_filings` | Filed reports, by state, year, entity name or 340B ID, entity type, kind of record or form. Short rows with the ClinX page and the official source; says plainly when a list is cut short. |
| `get_filing` | One filed report in full: every line as filed with its label, unit and amount, which claim counts are withheld and why, how the figures were checked, recorded discrepancies, and the official file's address, size and SHA-256. A very long report comes in parts. |
| `state_overview` | One place at a glance: a state's laws, its bills by stage, which group of reporting states it is in, how many filed reports ClinX holds for it, and what is next. `FED` gives the federal agencies. |
| `about_the_data` | Where the tables come from, when each was last brought up to date, what the filed reports table holds, what the three health center sets are and the day each was read, what HRSA's marks mean, what ClinX does not do, the licence. |

### The filed reports

The Scorecard at clinx.org holds the reports that covered entities and states file about 340B.
Three tools open it.

- Every figure is given exactly as the official file prints it. Columns ending `_as_filed` hold
  the printed text; `amount` and `claims` hold the same figure as a number, written as text just
  as the table has it. The server never rounds, estimates or fills in a figure.
- A claim count of 1 to 5 is never given, and neither is anything that would let a reader work
  one out. A row with a withheld count says `claims_withheld`, and the answer explains each
  reason. Every other figure on the row stays as filed. The server also holds back a small count
  that a file forgot to mark.
- The server ranks no one and calls no filer better or worse than another. It adds no sums.
  The three measures a report may carry (gross margin, external cost and the share) are the
  dataset's own, worked out from that one report's lines.
- A "filing" here is one record: a filer's own report, a row a state printed about one filer
  in its own report, or a state's totals. A filer's own report has a page on the site; a row a
  state printed, or a state's totals, points to the page of that state's reports.
- The tools are written for any form, kind of record and entity type the table holds. A value
  the code has not seen is returned as it is, with its own words.
- Ohio's reports are Excel workbooks the Department of Health posts on its own site. A workbook
  prints a 340B ID and no name, so `entity` is left out for an Ohio record and the name is given
  as `hrsa_name`, with the day of HRSA's list and a sentence saying it is HRSA's and not the
  filer's. A line carries `source_cell`, the workbook cell it was read from. A cell that holds
  words where a figure belongs has the words in `amount_as_filed` and no `amount`. `find_filings`
  finds an Ohio filer by HRSA's name or by its 340B ID.
- A report with many lines (the largest today has 1,030) comes in parts. One answer holds 100
  lines, or up to 150 when asked with `limit`, and fewer where the lines are long (see "How
  much one answer holds" below). The answer says how many lines are left and how to ask for the
  next part with `offset`, and it lists the report's schedules so that one schedule can be asked
  for.

Questions to try: "Which states require 340B reports, and which of them publish a report for
each hospital?" (`reporting_states`); "Find the 2025 reports filed in Rhode Island by health
centers" (`find_filings`); "What did the Idaho 2025 report print for total claims, and were
there any differences between its figures?" (`get_filing`).

### The health center tools

Five tools answer about health centers. Each reads files that ClinX built from a federal
agency's own files, and each answer names the agency's file and the day ClinX read it.

| Tool | What it answers |
| --- | --- |
| `find_health_centers` | The health centers that have records, by state, by words in the name, city or grant number, or by a year of report. Short rows in the order of the names, each with its key and its ClinX page. |
| `get_health_center_report` | What one health center reported to HRSA for one year. With no table named: the tables of that year, how many cells each has and how many hold a value or a mark. With a table named: every line of it with HRSA's titles and each value as published. A long table comes in parts. |
| `find_funding_opportunities` | HRSA's funding opportunities as Grants.gov lists them, by words, by status (forecasted or posted), only those of the Health Center Program, or those open on a day. With a number such as `HRSA-27-006`, that one in full. |
| `health_center_calendar` | The dated lines of the health center funding calendar between two days: a notice expected, an application due, an award expected, comments closing, a rule taking effect. Each row links the official page. |
| `shortage_area_changes` | Shortage designations and health center sites. With no state: the national counts. With a state: its counts by kind and status. With `show`: the designations that changed in the last 365 days, the ones proposed for withdrawal, or the sites added to scope. |

A few things about these answers are deliberate.

- **A value is given as HRSA published it**, character for character. Nothing is rounded, added
  up or estimated. A number HRSA's workbook holds with an exponent (`2e+05`) is given that way
  and marked as such.
- **A mark is given as the mark, with its meaning in words.** HRSA writes `-` where a cell has
  no entry, `--` where it withheld a small count of patients, and `---` where it withheld a
  value as confidential. The server passes each on and never works out what was withheld.
- **Some cells have no value on ClinX.** A count of patients from 1 to 15 is not repeated, and
  text a health center typed into a cell is not kept. The answer says which of the two it is.
- **Some years do not name a health center in every sheet.** Then the table is listed with the
  status `unnamed` and no value is given for it.
- **No tool puts health centers in the order of a figure,** and none compares one with another.
  A list is in the order of the names.
- **A forecast is a forecast.** A date or an amount on a forecasted opportunity is the agency's
  estimate, and the answer says so on every row.
- **No contact person.** The line of a funding notice that names whom to write to is never read
  into memory, so no tool can give it out.
- **Each health center file is its own piece.** If one cannot be read, the tool that needs it
  says so plainly and every other tool goes on answering.

Questions to try: "Which health centers in Rhode Island have records, and for which years?"
(`find_health_centers`); "What did that health center report in Table 3A for 2025?"
(`get_health_center_report`); "Which HRSA funding opportunities are open today, and when are
they due?" (`find_funding_opportunities`); "What health center funding dates fall in December?"
(`health_center_calendar`); "Which shortage designations in Rhode Island are proposed for
withdrawal?" (`shortage_area_changes`).

A bill at the stage `Did not pass` ended without becoming law; it is finished, and no tool shows
it as pending. The calendar has a kind `report due` for the day a state's report is due.

Every row in an answer carries the link to the official page it came from, and every answer says
the day its table was last brought up to date. The plain sentences about each law, bill and
notice were written with Claude, Anthropic's AI model, from the official text. They are a reading
aid, not legal advice, and a bill's sentence can trail a newer version of its text. Dates on the
calendar can move.

Three codes name a place. A state has its two letters, such as `RI`. Congress is `US`. Federal
agencies, which write rules and notices and not bills, are `FED`.

"Coming up" and "open for comment" go by the day the question is asked, not by the day the
tables were last brought up to date. A federal notice nobody has read yet is still listed, with
the Federal Register's own facts and no sentence; the answer says so.

A mistake in what was asked (a state that is not a two-letter code, a date in the wrong shape, a
date that cannot exist such as 2026-13-45, a bill that is not in the table) is said back in plain
words, so the assistant can correct itself. A wrong value longer than 80 characters is said back
as its first 80 and an ellipsis.

### How much one answer holds

A tool's answer is sent once, as JSON in one block of text. An assistant that takes too much at
once loses the thread, and directories of servers ask that a reply stay small, so one answer is
held to 60,000 characters, counted as it travels inside the reply.

A list keeps whole rows only: as many as fit, never part of a row and never part of a value. When
rows are left out for size, the answer says so in a sentence: how many are shown of how many
matched, and which filters narrow the question to reach the rest. `get_filing` and
`get_health_center_report` give a long report in parts and name the `offset` of the next part.
The counts in an answer (`matched`, `shown`, `lines_total`, `lines_returned`) are always the true
ones. No figure is changed, rounded or left half-written to make an answer fit.

## Try it

**The quick way.** Give an assistant the address `https://clinx.org/mcp` as a connector (an MCP
server) and ask it a question. Nothing to install.

**Your own copy.** You need [Node.js](https://nodejs.org) 20 or newer. Nothing else is
installed: the server uses no outside packages.

    git clone https://github.com/VVillBerchtold/clinx-mcp.git
    cd clinx-mcp
    npm test

**As a helper program for an assistant app.** Most apps take a small piece of configuration that
names the program to start. For Claude Desktop and similar apps:

```json
{
  "mcpServers": {
    "clinx-340b": {
      "command": "node",
      "args": ["/full/path/to/clinx-mcp/src/stdio.js"]
    }
  }
}
```

For Claude Code:

    claude mcp add clinx-340b -- node /full/path/to/clinx-mcp/src/stdio.js

**Over the web, on your own computer.**

    node src/serve.js

It listens at `http://127.0.0.1:8340/mcp`, on your computer only. Ask it something with `curl`:

    curl -s http://127.0.0.1:8340/mcp -H 'content-type: application/json' \
      -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"state_overview","arguments":{"state":"RI"}}}'

**From files on disk instead of clinx.org.** Set `CLINX_DATA_DIR` to a folder that holds
`340b-state-laws.json`, `340b-bills.json`, `340b-notices.json` and `340b-events.json`, and
if you want the filed reports, `340b-state-filings.json` and `340b-reporting-states.json`. The
last two are read on their own: if either is missing or cannot be read, the other nine tools
answer as before and the three new ones say the file could not be read.

**The filed reports are read one at a time.** The whole filed reports file is 24 MB, more than
a small web host will serve as one file. clinx.org also publishes `340b-filings-index.json`
(each report without its lines) and one file for each report, `filings/<filing id>.json`. The
server reads the index once an hour and a report's file only when `get_filing` asks for that
report, and keeps the last few for an hour. `src/split.js` cuts the whole file into those pieces
and the site calls it when it is built. If a folder or a host has the index, the server uses it;
if the index cannot be read it reads `340b-state-filings.json` whole and cuts it with the same
function, which is how the program on your own computer works with a plain copy of the data.
`src/pages.js` is the rule for where each report is shown on clinx.org.

## What is in this folder

| File | What it does |
| --- | --- |
| `src/core.js` | The server itself: reads one message, writes one answer. |
| `src/split.js` | Cuts the filed reports file into an index and one small file for each report. |
| `src/pages.js` | Where each filed report is shown on clinx.org. |
| `src/tools.js` | The list of all seventeen tools, and the eleven about 340B with `about_the_data`: what each takes and how it answers from the tables. |
| `src/health-center-tools.js` | The five health center tools, and what `about_the_data` says about the health center sets. |
| `src/health-centers.js` | The health center files: their names, the shape each is checked for, and when each is read. |
| `src/inputs.js` | Small helpers both groups of tools share for checking what was asked. |
| `src/tables.js` | Fetches the six 340B files and keeps them for an hour. Holds a report's lines as text and reads them back only for the report asked for. Hands the health center pieces to the tools. |
| `src/disk.js` | Reads the table files from a folder instead, for a program on your own computer. |
| `src/usage.js` | Describes a request by kind, for a host that wants to count what is asked. See below. |
| `src/stdio.js` | The door for a helper program on your own computer. |
| `src/http.js` | The door for the web. One request in, one answer out, nothing remembered. |
| `src/serve.js` | Runs the web door on your own computer. |
| `src/worker.js` | The web door shaped for an edge host that runs it by itself. Not deployed: clinx.org serves the same door from the site's own program ([clinx-org](https://github.com/VVillBerchtold/clinx-org), `worker/assistants.ts`). |
| `src/*.d.ts` | The shapes of the pieces above, for a host written in TypeScript. |
| `test/` | The tests, and small real slices of the tables (`test/data/`) they run on. |

## Which versions of the protocol it speaks

MCP changed shape in its revision of 28 July 2026. Before it, a client opened with a handshake
(`initialize`) and the two sides agreed a version once. Since it, there is no handshake: every
request names its version, and a client can ask `server/discover` what a server speaks.

Both kinds of client are in use, so the server answers both: the 2026-07-28 revision, and the
older 2025-11-25, 2025-06-18, 2025-03-26 and 2024-11-05. A request that names the newer version
is answered the newer way. Everything else is answered the older way.

Since version 0.3.3 the server also names its website and its icon, the ClinX mark as clinx.org
serves it (a PNG of 1024 by 1024 and the same mark as an SVG). The 2026-07-28 revision carries
them on every answer. Of the older revisions only 2025-11-25 has a place for them, in the
handshake; a client that agrees an earlier one is told the name, title and version as before.

On 15 November 2026 Claude Code 2.1.306 was pointed at the server. It opened with
`server/discover`, was told 2026-07-28, and listed the tools (seven then, seventeen now). Those exact messages are now
one of the tests. A whole question asked through an assistant, start to finish, has not been
tried yet; nor has any other client.

## Safety and privacy

- **Read-only.** No tool changes anything anywhere. Each tool says so to the client in its hints,
  as plain true or false: `readOnlyHint` true, `destructiveHint` false, `idempotentHint` true and
  `openWorldHint` false.
- **Nothing about the asker is kept.** No log of questions, no cookies, no sessions, no sign-in.
  Run on your own computer, the server counts nothing at all.
- **It cannot be turned on another site.** The only addresses it ever fetches are the
  public data files under clinx.org/data/. The small ones are fetched at most once an hour,
  and one health center's or one state's file only when a question names it, whatever it is asked.
- **Small requests only.** The web door takes `POST` with JSON, up to 64 KB, and nothing else.
- **On your own computer** it listens on `127.0.0.1` only and refuses calls from web pages that
  are not on that computer, so a page you visit cannot reach it.
- If clinx.org cannot be reached, a tool says so. If a refetch fails, the copy already held is
  still served.
- **No contact person.** The health center tools give no contact person's name, email address
  or phone number.

## Counting what is asked, without knowing who asked

Whoever hosts the server for others will want to know which questions people bring to it, so
the work goes where it is used. The web door can hand a host a short description of each
request (`counted`, in `src/http.js`). The server keeps nothing itself; the host decides where
the descriptions go.

A description has eight fields and never more:

| Field | Example | What it is |
| --- | --- | --- |
| `method` | `tools/call` | The kind of request. |
| `tool` | `find_bills` | The tool called, if one was. |
| `outcome` | `answered` | Answered, a mistake in what was asked, refused, or the tables could not be reached. |
| `place` | `TX` | The state asked about, `US` or `FED`. A territory's code is not kept. |
| `filters` | `kind=contract-pharmacy text` | The filters used, by name. |
| `looked_up` | `ri-2025-hb5634` | The public row a "get" tool found: a bill, a law, a notice or a filed report. A health center's key is not kept. |
| `client` | `claude-code` | The name the assistant program gives itself. |
| `version` | `2026-07-28` | The protocol version it asked for. |

What is left out matters more than what is kept:

- **Never the words.** Search words, dates and anything else typed freely are not kept. A
  filter is kept by name only (`text`), and its value only when it is one of the tool's own
  listed choices (`kind=contract-pharmacy`).
- **Never the asker.** No internet address, no browser or device description, nothing that
  could tell one asker from another.
- **A looked-up id comes from the answer**, not from what was typed: it is a public row the
  server found and gave back.
- A client's name is cut down to plain letters and forty characters.

`test/usage.test.js` checks each of these with a made-up name, number and email address typed
into every field, and fails if any of it comes out the other side.

The copy at clinx.org/mcp is counted this way and no other. The site's
[Privacy page](https://clinx.org/privacy/#assistants) says the same to its readers.

## Tests

    npm test          # 101 tests and 3 live checks that are skipped, no network
    npm run test:live # three checks against the published files at clinx.org

The tests cover both protocol eras, every tool, every kind of mistaken input, a health center
file that cannot be read (its tool says so and the others go on answering), what a host is
told about a request and what it is not, the helper program
started and driven as an app would drive it, and the web door both as a function and as a real
server on a local port.

## What is not done

- It is not published as a package, so there is no one-line install.
- For 340B it answers from the state law, bills, federal notices, calendar, filed reports and
  reporting states tables. The registry tables are not offered yet, and the drug rows of a report are
  given as lines of that report, not as a separate search.
- Federal notices go back to January 2025. Court cases, and which drug makers an agency has
  approved for the rebate pilot, are not in the tables.
- Health center records are held for the states ClinX has read so far; `find_health_centers`
  says which. A health center's sites and the ZIP codes of its patients are on its ClinX page
  and in its file, and no tool gives them yet. The Federal Register documents of the funding
  calendar are given as dated lines, not as a search of their own.
- Hearings appear once a legislature posts an agenda that names a listed bill. In November 2026,
  with most legislatures out of session, there are none.

## Licence

The code is under the [MIT licence](LICENSE). The tables it reads are under Creative Commons
Attribution 4.0; credit them as "ClinX, clinx.org, retrieved <date>, CC BY 4.0".
