# Budget KPI Cards SVG — DAX Measure

Generates the SVG version of the Budget KPI card. Renders through an image/HTML visual set to the `Image URL` data category, and rebuilds itself from `[Actual]` and `[Budget]` on every filter change.

## What it does

- Pulls the current year off `'Date'[Year]`, `[Actual]`, and `[Budget]`.
- Passes those into `BudgetKPICardSVG(...)`, a helper measure that builds the card markup (bar, badge, labels) at a fixed 450×240 coordinate space.
- Wraps the result in an `<svg>` tag with `viewBox`, so it scales cleanly to whatever container the image visual gives it.

## Config

- `CurrencySymbol` — change `"$"` to `"€"`, `"£"`, `"¥"`, etc. to switch currency display.
- `CardWidth` / `CardHeight` — the internal coordinate space (`viewBox`), matched to the original `Budget Card.dc.html` design spec. Changing these reflows the card's internal layout, not just its rendered size — rendered size is controlled by the visual's container.

## Code

```dax
Budget KPI Cards SVG = -- CONFIG: change this to switch currency, e.g. "€", "£", "¥"
VAR CurrencySymbol = "$"

-- Card proportions from Budget Card.dc.html design spec -- this IS the SVG's internal coordinate space via viewBox
VAR CardWidth = 450
VAR CardHeight = 240

VAR ActualCY = [Actual]
VAR BudgetCY = [Budget]
VAR CurrentYear = MAX ( 'Date'[Year] )
VAR Card =
    BudgetKPICardSVG ( 0, 0, CardWidth, CardHeight, "FY " & CurrentYear, ActualCY, BudgetCY, CurrencySymbol )
VAR _Prefix =
    "<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 " & CardWidth & " " & CardHeight & "' width='100%' height='100%' preserveAspectRatio='xMidYMid meet'>"
VAR _Suffix =
    "</svg>"
RETURN
    _Prefix & Card & _Suffix
```

## Dependencies

- `BudgetKPICardSVG` — extension measure that returns the card's inner SVG markup. Must exist in the model for this measure to resolve.
- `[Actual]`, `[Budget]` — base measures.
- `'Date'[Year]` — active year column in the date table.

## Usage

1. Add this measure to a card or image visual.
2. Set the visual's field to `Image URL` data category if not inherited from the measure's formatting.
3. Filter by year (or any date context) — the card regenerates for whatever `'Date'[Year]` resolves to in the current filter.****
