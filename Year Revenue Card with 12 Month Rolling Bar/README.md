![Yearly Revenue Card preview](preview.png)

# Yearly Revenue Card SVG — DAX Measure

Rolling 12-month revenue card: big current-year total, % change vs. prior year, and a bar-in-bar chart comparing current year (bars) against prior year (dashed lines) per month.

Renders through an image/HTML visual set to the `Image URL` data category, and regenerates from `[Total Revenue]` and `[Previous Year Revenue]` on every filter change.

## What it does

- Summarizes `'Date'[Month Number]` / `'Date'[Month]` for the current filter context, pulling `CurrentYearValue` and `PastYearValue` per month.
- Computes the year total, the year-over-year % change, and picks a color for that change (green if up, red if down).
- Draws one bar per month for the current year, sized off the max value across both years so the two series share a scale. Each bar is colored blue if it beats the prior year, red if it doesn't.
- Draws a dashed horizontal line per month marking the prior year's value at that point, so the comparison sits right on top of the bar.
- Labels each month with its first letter along the bottom axis.
- Wraps everything in a card with a drop shadow, title, and big value at the top.

## Config

All at the top of the measure, under `CONFIG`:

| Variable | What it controls |
|---|---|
| `_CYMeasure` | Label only — not wired to a dynamic measure reference, just documents which measure feeds the card (`Total Revenue`) |
| `_CurrencySymbol` | Currency prefix on the big value, e.g. `"$"`, `"€"`, `"£"` |
| `_BlueColor` / `_RedColor` | Bar color when current year is ahead / behind prior year |
| `_DashColor` | Color of the prior-year comparison line |
| `_TitleColor`, `_ValueColor`, `_SubLabelColor`, `_AxisLabelColor` | Text colors for the title, big number, "vs. prior year" subtext, and month labels |
| `_ShowCard` | `TRUE`/`FALSE` — toggles the white card background, border, and shadow |
| `_CardFill` / `_CardStroke` | Card background and border color, only used if `_ShowCard = TRUE` |
| `_AxisRoundTo` | Rounds the Y-axis max up to the nearest multiple of this number (default `50`), so bar heights land on clean scale increments |
| `_ShowRevenueIcon` | `TRUE`/`FALSE` — toggles the icon pulled from the separate `[SVG Icon - Revenue]` measure |

## Code

```dax
Yearly Revenue Card SVG = 
-- ====================== CONFIG ======================
VAR _CYMeasure        = "Total Revenue"
VAR _CurrencySymbol   = "$"
VAR _BlueColor        = "#0E64C5"
VAR _RedColor         = "#F4525A"
VAR _DashColor        = "#111827"
VAR _TitleColor       = "#374151"
VAR _ValueColor       = "#111827"
VAR _SubLabelColor    = "#9CA3AF"
VAR _AxisLabelColor   = "#888888"
VAR _ShowCard         = TRUE
VAR _CardFill         = "#ffffff"
VAR _CardStroke       = "#e5e7eb"
VAR _AxisRoundTo      = 50

VAR _ShowRevenueIcon  = TRUE
-- ====================================================

VAR MonthData =
    ADDCOLUMNS (
        SUMMARIZE (
            'Date',
            'Date'[Month Number],
            'Date'[Month]
        ),
        "CurrentYearValue", [Total Revenue],
        "PastYearValue", [Previous Year Revenue]
    )

VAR MaxValueCY =
    MAXX ( MonthData, [CurrentYearValue] )

VAR MaxValuePY =
    MAXX ( MonthData, [PastYearValue] )

VAR MaxValue =
    MAX ( MaxValueCY, MaxValuePY )

VAR YAxisMax =
    CEILING ( MaxValue * 1.1, _AxisRoundTo )

VAR SafeYAxisMax =
    IF ( YAxisMax > 0, YAxisMax, 1 )

VAR _CYTotal =
    SUMX ( MonthData, [CurrentYearValue] )

VAR _PYTotal =
    SUMX ( MonthData, [PastYearValue] )

VAR _ChangePct =
    DIVIDE ( _CYTotal - _PYTotal, _PYTotal )

VAR _ChangeColor =
    IF ( _ChangePct >= 0, "#16a34a", "#dc2626" )

VAR _ChangeSign =
    IF ( _ChangePct >= 0, "+", "" )

VAR _CurrentYearLabel =
    "Current - " & [Current Year Text]

VAR _PriorYearLabel =
    [Previous Year Text]

-- Revenue Icon from separate measure
VAR _RevenueIcon =
    IF ( _ShowRevenueIcon, [SVG Icon - Revenue], "" )

-- Coordinate space
VAR HeaderHeight  = 115
VAR ChartHeight   = 250
VAR ChartWidth    = 650
VAR TotalHeight   = HeaderHeight + ChartHeight

VAR PaddingTop    = HeaderHeight + 10
VAR PaddingRight  = 15
VAR PaddingBottom = 45
VAR PaddingLeft   = 25

VAR PlotWidth =
    ChartWidth - PaddingLeft - PaddingRight

VAR PlotHeight =
    TotalHeight - PaddingTop - PaddingBottom

VAR XStep =
    DIVIDE ( PlotWidth, 12 )

VAR YScale =
    DIVIDE ( PlotHeight, SafeYAxisMax )

VAR BaselineY =
    TotalHeight - PaddingBottom

VAR BarWidth =
    XStep * 0.55

VAR _Prefix =
    "data:image/svg+xml;utf8,<svg viewBox='0 0 " & ChartWidth & " " & TotalHeight &
    "' preserveAspectRatio='xMidYMid meet' xmlns='http://www.w3.org/2000/svg'>"

VAR _Defs =
    "<defs><filter id='sh' x='-5%' y='-5%' width='115%' height='130%'><feDropShadow dx='0' dy='3' stdDeviation='4' flood-color='#00000018'/></filter></defs>"

VAR _Card =
    IF (
        _ShowCard,
        "<rect x='4' y='4' width='" & ( ChartWidth - 8 ) &
        "' height='" & ( TotalHeight - 8 ) &
        "' rx='18' fill='" & _CardFill &
        "' stroke='" & _CardStroke &
        "' stroke-width='1.5' filter='url(#sh)'/>",
        ""
    )

-- Top Text
VAR _Title =
    "<text x='" & ( ChartWidth / 2 ) &
    "' y='30' font-family='Arial' font-size='20' fill='" &
    _TitleColor &
    "' font-weight='500' text-anchor='middle'>" &
    _CurrentYearLabel &
    "</text>"

-- Main Value
VAR _BigValue =
    "<text x='" & ( ChartWidth / 2 ) &
    "' y='78' font-family='Arial' font-size='46' fill='" &
    _ValueColor &
    "' font-weight='700' text-anchor='middle'>" &
    _CurrencySymbol & FORMAT ( _CYTotal, "#,##0" ) &
    "</text>"

-- Change Text
VAR _ChangeText =
    "<text x='" & ( ChartWidth / 2 ) &
    "' y='102' font-family='Arial' font-size='17' text-anchor='middle'>" &
        "<tspan fill='" & _ChangeColor & "' font-weight='700'>" &
            _ChangeSign & FORMAT ( _ChangePct, "0.0%" ) &
        "</tspan>" &
        "<tspan fill='" & _SubLabelColor & "'> vs. " & _PriorYearLabel & "</tspan>" &
    "</text>"

VAR CYColumns =
    CONCATENATEX (
        FILTER (
            MonthData,
            NOT ( ISBLANK ( [CurrentYearValue] ) )
        ),
        VAR CX =
            PaddingLeft + ( ( [Month Number] - 0.5 ) * XStep )

        VAR BarH =
            [CurrentYearValue] * YScale

        VAR BarY =
            BaselineY - BarH

        VAR BarX =
            CX - ( BarWidth / 2 )

        VAR BarColor =
            IF (
                [CurrentYearValue] >= [PastYearValue],
                _BlueColor,
                _RedColor
            )

        RETURN
            "<rect x='" & BarX &
            "' y='" & BarY &
            "' width='" & BarWidth &
            "' height='" & BarH &
            "' rx='3' fill='" & BarColor & "' />",
        " ",
        [Month Number],
        ASC
    )

VAR PYDashes =
    CONCATENATEX (
        FILTER (
            MonthData,
            NOT ( ISBLANK ( [PastYearValue] ) )
        ),
        VAR CX =
            PaddingLeft + ( ( [Month Number] - 0.5 ) * XStep )

        VAR DashY =
            BaselineY - ( [PastYearValue] * YScale )

        VAR DashX1 =
            CX - ( BarWidth / 2 ) - 3

        VAR DashX2 =
            CX + ( BarWidth / 2 ) + 3

        RETURN
            "<line x1='" & DashX1 &
            "' y1='" & DashY &
            "' x2='" & DashX2 &
            "' y2='" & DashY &
            "' stroke='" & _DashColor &
            "' stroke-width='3' />",
        " ",
        [Month Number],
        ASC
    )

VAR MonthLabels =
    CONCATENATEX (
        MonthData,
        VAR CX =
            PaddingLeft + ( ( [Month Number] - 0.5 ) * XStep )

        VAR LY =
            BaselineY + 22

        RETURN
            "<text x='" & CX &
            "' y='" & LY &
            "' font-family='Arial' font-size='14' fill='" &
            _AxisLabelColor &
            "' text-anchor='middle'>" &
            LEFT ( [Month], 1 ) &
            "</text>",
        " ",
        [Month Number],
        ASC
    )

VAR _Suffix =
    "</svg>"

RETURN
    _Prefix &
    _Defs &
    _Card &
    _RevenueIcon &
    _Title &
    _BigValue &
    _ChangeText &
    CYColumns &
    PYDashes &
    MonthLabels &
    _Suffix
```

## Dependencies

- `[Total Revenue]` — current year revenue measure, referenced per month via `SUMMARIZE`.
- `[Previous Year Revenue]` — prior year revenue measure, same-period comparison.
- `[Current Year Text]`, `[Previous Year Text]` — measures returning the display labels used in the title and subtext (e.g. "2026", "2025").
- `[SVG Icon - Revenue]` — separate measure returning icon markup, only pulled in if `_ShowRevenueIcon = TRUE`.
- `'Date'[Month Number]`, `'Date'[Month]` — active date table columns used to build the 12-month axis.

## Usage

1. Add this measure to a card or image visual.
2. Set the visual's field to `Image URL` data category if it isn't inherited from the measure's formatting.
3. Filter by year — the card recalculates the current-year total, comparison %, and all 12 bars for whatever year is in context.
4. To reskin colors or toggle the card background/icon, edit the `CONFIG` block at the top — no need to touch the layout logic below it.

## Notes

- Bar height scales off `SafeYAxisMax`, which is the higher of the two years' max monthly value, rounded up to the nearest `_AxisRoundTo`. This keeps both years on one shared scale so the comparison is visually honest.
- Month labels only show the first letter (J, F, M...) — intentional for a compact 650×365 card. If you need full month abbreviations, `MonthLabels` is the section to edit.
- `_CYMeasure` is documentation only, not a live reference — if you swap the underlying revenue measure, update both `_CYMeasure` (label) and the actual `[Total Revenue]` calls in `MonthData`.
