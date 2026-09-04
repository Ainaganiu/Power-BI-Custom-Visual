# Bar Chart Race

An animated ranked bar chart for Power BI that shows how categories rise and fall over time — the "racing bars" format popularized on YouTube, built as a fully themable, keyboard-accessible custom visual.

This is Pbiviz Visual

![Bar Chart Race in action](https://github.com/Ainaganiu/Custom-Visual/blob/main/Race%20Bar%20Chart/Bar%20Chart%20Image.png?raw=true)

## What it does

Bar Chart Race animates a ranked list of categories across periods (months, years, quarters — any numeric sort key). Bars grow, shrink, and reorder as the animation plays, with a running leader subtitle, rank-change badges, an optional cumulative mode, and a scrubbable timeline you can drag to any point in the story.

## How to use it

1. **Add the visual** to your report canvas from the Visualizations pane.
2. **Bind your fields:**
   - **Category** — the racer label (e.g. Product, Country, Team)
   - **Period** — a numeric sort key that drives the animation order (e.g. Month Number, Year)
   - **Value** — the measure that sets bar length (e.g. Sum of Sales)
   - Optional: **Group** for a subtitle under each bar, **Period Label** for readable period text, **Tooltips** for extra hover fields
3. **Press play** using the header button, or drag the timeline handle to scrub manually.
4. **Customize** in the Format pane — every color defaults to your report theme, so most reports need no changes at all. Adjust playback speed, Top N, cumulative mode, fonts, and the tooltip style whenever you want something different.

## Highlights

- Smooth, themable animation with adjustable speed, easing, and playback direction (forward, reverse, ping-pong)
- Cumulative or per-period racing modes, with a running-total indicator
- Rank-change badges (▲ ▼ ● ★) with independent colors
- Custom tooltip card with a live percent-of-total bar, or switch to the native Power BI tooltip
- Cross-filtering — click a bar to filter the rest of the report
- Full keyboard navigation and high-contrast mode support
- Native font controls on every text element
