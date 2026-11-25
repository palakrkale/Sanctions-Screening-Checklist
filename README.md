# Sanctions Screening Checklist

A sanctions-screening checklist built in Google Sheets, using the public **OFAC SDN (Specially Designated Nationals)** list. You enter a name, and the sheet automatically checks it against the SDN list, scores how confident the match is, and flags whether it needs a closer look.

This is a practice project demonstrating a basic AML/KYC watchlist-screening workflow: automated name matching plus a human decision layer (**Clear / Review / Escalate**) on top.

🔗 [View the Project here](https://docs.google.com/spreadsheets/d/1aPa-UxMlEplxlRLANxN4eCehPxR5BGWDuxbCgaQhnxA/edit?usp=sharing)

---

## Sheet structure

| Tab | Purpose |
| --- | --- |
| `SDN_List` | The OFAC sanctions list, imported as-is|
| `Screening_Queue` | Where you enter a name to screen and see the match result and decision |

## Screening_Queue

| Col | Name | What it shows |
| --- | --- | --- |
| A | Name Entered | The name you type |
| B | Match Found? | Yes / Maybe / No, derived from Match % |
| C | Match % | 100 / 75 / 50 / 0, from the `MATCHSCORE` script |
| D | Matched SDN Name | The closest matching name on the SDN list, or "No Match" |
| E | Program | The sanctions program tied to that matched name (e.g. SDGT) |
| F | Status | Manual Dropdown: Clear / Review / Escalate |
| G | Notes | Free-text reasoning for the Status decision |

## How MATCHSCORE works

A custom Apps Script function, `MATCHSCORE`, compares the entered name against the matched SDN name in four tiers:

| Tier | Condition | Score |
| --- | --- | --- |
| 1 | Exact match | 100% |
| 2 | Whole-word match | 75% |
| 3 | Partial substring match | 50% |
| 4 | Weak/fallback match | 25% |
| — | No match found | 0% |

Match % then drives Match Found? automatically:

- **100%** → Yes
- **50–75%** → Maybe
- **0% (or below 50%)** → No

## How to use it

1. Go to the `Screening_Queue` tab.
2. Type a name into column A (Name Entered).
3. Columns B–E fill in automatically: Match %, Matched SDN Name, Program, and Match Found?.
4. Review the result and set a Status in column F: Clear, Review, or Escalate (100% matches should always be Escalate).
5. Write a short justification in column G (Notes) explaining the match and the decision.

## Automation (Apps Script)

A small Apps Script project (`Screening_queue_automation`) runs behind the sheet:

- **`onEdit(e)`** — watches column A. If Name Entered is cleared, it automatically clears the Status dropdown in that row, so a blank row never carries a stale decision.
- **`MATCHSCORE(name, sdnName)`** — the custom function powering the Match % column, described above.

## Status & Notes

Status and Notes are the two columns left to human judgment. Status is the actual decision (Clear / Review / Escalate), and Notes is the short reasoning behind it: what matched, how confident it is, and what action was taken or is needed next.

## Limitations

This is a learning/portfolio project, not a production compliance tool. The matching logic is simple string comparison (exact, whole-word, substring), and the SDN list is a static snapshot rather than a live feed. A real screening system would need regular list updates, more robust name-matching, and a documented audit trail of decisions.
