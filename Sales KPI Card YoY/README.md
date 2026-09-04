![Sales Card preview](preview.png)

**From sketch to card:**

| Sketch | Final output |
|---|---|
| ![Original sketch](sketch.jpeg) | ![Final rendered card](final-output.png) |


# KPI SVG - Sales Card — DAX Measure

Full sales KPI card: current year total with YoY change arrow, prior-year comparison, top product with a segment-share progress bar, and a quarterly revenue bar chart — all in one SVG.

Renders through an image/HTML visual set to the `Image URL` data category, and rebuilds from `[Total Revenue]`, `[Previous Year Revenue]`, `[Revenue YoY %]`, `[Top Product Name]`, and `[Top Product Segment %]` on every filter change.

Started as a hand sketch on my tablet — sales figure and YoY arrow up top, top product with a segment % bar, quarterly bars along the bottom. Every element in this measure maps back to something on that sketch.

## What it does

- **Header value** — current year revenue as the big number, sized down automatically if the formatted text gets long (`_ValueFontSize` switches between 64/56/50pt based on character count), so a 7-figure number doesn't overflow the card.
- **YoY indicator** — arrow (↗/↘) plus signed percentage next to the main value, green if current year beats prior year, red if it doesn't. Its X position shifts left as the main value gets wider, so it never overlaps.
- **Last year line** — plain comparison text under the header value.
- **Top product block** — pulls the top product name, escapes `&` and `<` so it won't break the SVG if the product name contains them, truncates to 19 characters with `...` if longer, and shows it uppercase.
- **Segment % progress bar** — a light-gold track with a solid-gold fill sized to the top product's share of segment revenue.
- **Quarterly bar chart** — four bars (Q1–Q4) for the current year (`MAX('Date'[Year])`), scaled against an axis max rounded up to the nearest 10,000. Includes gridlines, Y-axis value labels, and Q1–Q4 labels on the X-axis.
- Wraps the whole thing in a rounded card with a drop shadow and border, viewBox `0 0 1200 650`.

## Config

All at the top of the measure:

| Variable | What it controls |
|---|---|
| `_CurrencySymbol` | Currency prefix on the main value, e.g. `"$"`, `"€"`, `"£"` |
| `_BlueColor` | Main value text color |
| `_GreenColor` / `_RedColor` | YoY indicator color, up vs. down |
| `_TextDark` / `_TextGrey` | Header/label text vs. secondary text (Last Year line) |
| `_GridColor` | Chart gridline color |
| `_GoldColor` / `_GoldLight` | Segment % bar fill / track color |
| `_BarColor` | Quarterly bar fill color |
| `_CardStroke` / `_CardFill` | Card border and background color |

## Code

```dax
KPI SVG - Sales Card = 
VAR _CurrencySymbol = "$"

VAR _BlueColor = "#0E4A7B"
VAR _GreenColor = "#148A55"
VAR _RedColor = "#D64545"
VAR _TextDark = "#2F3542"
VAR _TextGrey = "#777777"
VAR _GridColor = "#D7D7D7"
VAR _GoldColor = "#D99812"
VAR _GoldLight = "#F4EAD8"
VAR _BarColor = "#2F74A8"
VAR _CardStroke = "#D9D9D9"
VAR _CardFill = "#FFFFFF"

VAR _CY =
    COALESCE ( [Total Revenue], 0 )

VAR _PY =
    COALESCE ( [Previous Year Revenue], 0 )

VAR _YoY =
    COALESCE ( [Revenue YoY %], 0 )

VAR _YoYColor =
    IF ( _CY >= _PY, _GreenColor, _RedColor )

VAR _YoYArrow =
    IF ( _CY >= _PY, "↗", "↘" )

VAR _CYText =
    _CurrencySymbol & FORMAT ( _CY, "#,##0" )

VAR _PYText =
    "Last Year: " & _CurrencySymbol & FORMAT ( _PY, "#,##0" )

VAR _YoYText =
    _YoYArrow & " " & FORMAT ( _YoY, "+0%;-0%;0%" ) & " YoY"

VAR _ValueFontSize =
    SWITCH (
        TRUE (),
        LEN ( _CYText ) >= 12, 50,
        LEN ( _CYText ) >= 10, 56,
        64
    )

VAR _YoYX =
    SWITCH (
        TRUE (),
        LEN ( _CYText ) >= 12, 520,
        LEN ( _CYText ) >= 10, 470,
        420
    )

VAR _TopProductRaw =
    COALESCE ( [Top Product Name], "N/A" )

VAR _TopProductClean1 =
    SUBSTITUTE ( _TopProductRaw, "&", "&amp;" )

VAR _TopProductClean2 =
    SUBSTITUTE ( _TopProductClean1, "<", "&lt;" )

VAR _TopProductShort =
    IF (
        LEN ( _TopProductClean2 ) > 19,
        LEFT ( _TopProductClean2, 19 ) & "...",
        _TopProductClean2
    )

VAR _TopProductText =
    "TOP PRODUCT: " & UPPER ( _TopProductShort )

VAR _SegmentPct =
    COALESCE ( [Top Product Segment %], 0 )

VAR _SegmentPctText =
    FORMAT ( _SegmentPct, "0%" )

VAR _ProgressFullWidth = 320

VAR _ProgressWidth =
    IF (
        _SegmentPct > 0,
        MIN ( _ProgressFullWidth, MAX ( 4, _ProgressFullWidth * _SegmentPct ) ),
        0
    )

VAR _CurrentYear =
    MAX ( 'Date'[Year] )

VAR _Q1 =
    COALESCE (
        CALCULATE (
            [Total Revenue],
            'Date'[Year] = _CurrentYear,
            'Date'[Quarter] = "Q1"
        ),
        0
    )

VAR _Q2 =
    COALESCE (
        CALCULATE (
            [Total Revenue],
            'Date'[Year] = _CurrentYear,
            'Date'[Quarter] = "Q2"
        ),
        0
    )

VAR _Q3 =
    COALESCE (
        CALCULATE (
            [Total Revenue],
            'Date'[Year] = _CurrentYear,
            'Date'[Quarter] = "Q3"
        ),
        0
    )

VAR _Q4 =
    COALESCE (
        CALCULATE (
            [Total Revenue],
            'Date'[Year] = _CurrentYear,
            'Date'[Quarter] = "Q4"
        ),
        0
    )

VAR _MaxQuarterValue =
    MAXX (
        {
            _Q1,
            _Q2,
            _Q3,
            _Q4
        },
        [Value]
    )

VAR _AxisMax =
    IF (
        _MaxQuarterValue <= 0,
        10000,
        CEILING ( _MaxQuarterValue, 10000 )
    )

VAR _ChartTop = 285
VAR _ChartBottom = 540
VAR _ChartHeight = _ChartBottom - _ChartTop
VAR _BaseY = _ChartBottom

VAR _Q1Height =
    DIVIDE ( _Q1, _AxisMax ) * _ChartHeight

VAR _Q2Height =
    DIVIDE ( _Q2, _AxisMax ) * _ChartHeight

VAR _Q3Height =
    DIVIDE ( _Q3, _AxisMax ) * _ChartHeight

VAR _Q4Height =
    DIVIDE ( _Q4, _AxisMax ) * _ChartHeight

VAR _Q1Y = _BaseY - _Q1Height
VAR _Q2Y = _BaseY - _Q2Height
VAR _Q3Y = _BaseY - _Q3Height
VAR _Q4Y = _BaseY - _Q4Height

VAR _Y1 = _AxisMax * 0.2
VAR _Y2 = _AxisMax * 0.4
VAR _Y3 = _AxisMax * 0.6
VAR _Y4 = _AxisMax * 0.8
VAR _Y5 = _AxisMax

VAR _Y1Pos = _BaseY - ( 0.2 * _ChartHeight )
VAR _Y2Pos = _BaseY - ( 0.4 * _ChartHeight )
VAR _Y3Pos = _BaseY - ( 0.6 * _ChartHeight )
VAR _Y4Pos = _BaseY - ( 0.8 * _ChartHeight )
VAR _Y5Pos = _BaseY - ( 1.0 * _ChartHeight )

VAR _SVG =
"
<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 1200 650' width='1200' height='650'>

    <defs>
        <filter id='shadow' x='-20%' y='-20%' width='140%' height='140%'>
            <feDropShadow dx='0' dy='10' stdDeviation='10' flood-color='#000000' flood-opacity='0.16'/>
        </filter>
    </defs>

    <rect x='25' y='25' width='1150' height='600' rx='28' fill='" & _CardFill & "' filter='url(#shadow)'/>
    <rect x='25' y='25' width='1150' height='600' rx='28' fill='none' stroke='" & _CardStroke & "' stroke-width='2'/>

    <!-- LEFT HEADER -->
    <text x='80' y='95' font-family='Segoe UI, Arial' font-size='38' font-weight='700' fill='" & _TextDark & "'>SALES</text>

    <!-- MAIN VALUE -->
    <text x='80' y='165' font-family='Segoe UI, Arial' font-size='" & _ValueFontSize & "' font-weight='800' fill='" & _BlueColor & "'>" & _CYText & "</text>

    <!-- YOY -->
    <text x='" & _YoYX & "' y='158' font-family='Segoe UI, Arial' font-size='28' font-weight='700' fill='" & _YoYColor & "'>" & _YoYText & "</text>

    <!-- LAST YEAR -->
    <text x='80' y='215' font-family='Segoe UI, Arial' font-size='25' font-weight='400' fill='" & _TextGrey & "'>" & _PYText & "</text>

    <!-- RIGHT PRODUCT SECTION -->
    <text x='800' y='85' font-family='Segoe UI, Arial' font-size='21' font-weight='700' fill='" & _TextDark & "'>" & _TopProductText & "</text>

    <text x='800' y='128' font-family='Segoe UI, Arial' font-size='25' font-weight='600' fill='" & _GoldColor & "'>Segment %</text>
    <text x='1120' y='128' text-anchor='end' font-family='Segoe UI, Arial' font-size='25' font-weight='700' fill='" & _GoldColor & "'>" & _SegmentPctText & "</text>

    <rect x='800' y='150' width='" & _ProgressFullWidth & "' height='34' rx='7' fill='" & _GoldLight & "'/>
    <rect x='800' y='150' width='" & _ProgressWidth & "' height='34' rx='7' fill='" & _GoldColor & "'/>

    <!-- CHART GRID -->
    <line x1='250' y1='" & _Y5Pos & "' x2='1110' y2='" & _Y5Pos & "' stroke='" & _GridColor & "' stroke-width='1.2'/>
    <line x1='250' y1='" & _Y4Pos & "' x2='1110' y2='" & _Y4Pos & "' stroke='" & _GridColor & "' stroke-width='1.2'/>
    <line x1='250' y1='" & _Y3Pos & "' x2='1110' y2='" & _Y3Pos & "' stroke='" & _GridColor & "' stroke-width='1.2'/>
    <line x1='250' y1='" & _Y2Pos & "' x2='1110' y2='" & _Y2Pos & "' stroke='" & _GridColor & "' stroke-width='1.2'/>
    <line x1='250' y1='" & _Y1Pos & "' x2='1110' y2='" & _Y1Pos & "' stroke='" & _GridColor & "' stroke-width='1.2'/>

    <!-- AXIS -->
    <line x1='250' y1='" & _Y5Pos & "' x2='250' y2='540' stroke='#222222' stroke-width='2'/>
    <line x1='250' y1='540' x2='1110' y2='540' stroke='#222222' stroke-width='2'/>

    <!-- Y AXIS LABELS -->
    <text x='225' y='" & _Y1Pos + 8 & "' text-anchor='end' font-family='Segoe UI, Arial' font-size='23' fill='#222222'>" & FORMAT ( _Y1, "#,##0" ) & "</text>
    <text x='225' y='" & _Y2Pos + 8 & "' text-anchor='end' font-family='Segoe UI, Arial' font-size='23' fill='#222222'>" & FORMAT ( _Y2, "#,##0" ) & "</text>
    <text x='225' y='" & _Y3Pos + 8 & "' text-anchor='end' font-family='Segoe UI, Arial' font-size='23' fill='#222222'>" & FORMAT ( _Y3, "#,##0" ) & "</text>
    <text x='225' y='" & _Y4Pos + 8 & "' text-anchor='end' font-family='Segoe UI, Arial' font-size='23' fill='#222222'>" & FORMAT ( _Y4, "#,##0" ) & "</text>
    <text x='225' y='" & _Y5Pos + 8 & "' text-anchor='end' font-family='Segoe UI, Arial' font-size='23' fill='#222222'>" & FORMAT ( _Y5, "#,##0" ) & "</text>

    <!-- BARS -->
    <rect x='305' y='" & _Q1Y & "' width='95' height='" & _Q1Height & "' rx='6' fill='" & _BarColor & "' opacity='0.95'/>
    <rect x='520' y='" & _Q2Y & "' width='95' height='" & _Q2Height & "' rx='6' fill='" & _BarColor & "' opacity='0.95'/>
    <rect x='735' y='" & _Q3Y & "' width='95' height='" & _Q3Height & "' rx='6' fill='" & _BarColor & "' opacity='0.95'/>
    <rect x='950' y='" & _Q4Y & "' width='95' height='" & _Q4Height & "' rx='6' fill='" & _BarColor & "' opacity='0.95'/>

    <!-- X LABELS -->
    <text x='352.5' y='585' text-anchor='middle' font-family='Segoe UI, Arial' font-size='24' fill='#111111'>Q1</text>
    <text x='567.5' y='585' text-anchor='middle' font-family='Segoe UI, Arial' font-size='24' fill='#111111'>Q2</text>
    <text x='782.5' y='585' text-anchor='middle' font-family='Segoe UI, Arial' font-size='24' fill='#111111'>Q3</text>
    <text x='997.5' y='585' text-anchor='middle' font-family='Segoe UI, Arial' font-size='24' fill='#111111'>Q4</text>

</svg>
"

RETURN
"data:image/svg+xml;utf8," & _SVG
```

## Dependencies

- `[Total Revenue]` — current year total, also called per-quarter via `CALCULATE` for the bar chart.
- `[Previous Year Revenue]` — prior year comparison figure.
- `[Revenue YoY %]` — year-over-year % measure (drives the arrow, color, and YoY text).
- `[Top Product Name]` — text measure for the top-product label.
- `[Top Product Segment %]` — the top product's share of segment revenue, drives the progress bar.
- `'Date'[Year]`, `'Date'[Quarter]` — active date table columns, used to isolate the current year and split it into Q1–Q4.

## Usage

1. Add this measure to a card or image visual.
2. Set the visual's field to `Image URL` data category if it isn't inherited from the measure's formatting.
3. Filter by year — quarterly bars, YoY comparison, and the top product block all recalculate for whatever year lands in `MAX('Date'[Year])`.
4. To reskin, edit the color/currency variables at the top — the layout math below them doesn't need to change.

## Notes

- `_MaxQuarterValue` uses `MAXX` over a literal table constructor `{ _Q1, _Q2, _Q3, _Q4 }` with `[Value]` — this is the standard DAX pattern for taking a max across scalar variables, not a typo.
- If a product name contains `&` or `<`, it gets HTML-escaped before insertion — without this, those characters would break the SVG markup. If you start seeing other special characters in product names causing render issues, extend the `SUBSTITUTE` chain the same way.
- The card is a fixed 1200×650 canvas with no `viewBox` scaling behavior applied beyond the tag itself — if you need it to scale responsively inside a smaller visual, add `preserveAspectRatio='xMidYMid meet'` to the `<svg>` tag, matching the pattern used in the other two card measures.
