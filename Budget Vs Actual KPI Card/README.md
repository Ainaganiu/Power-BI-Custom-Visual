# Three KPI card builds, same design: what actually changes for the person using the report

FY budget vs. limit, % used, red flag when you're over. Built three different ways. Here's what changes once someone's actually clicking around the report, not just looking at a screenshot of it.

This visual is inspired by [Zsolt Szabó](https://www.linkedin.com/in/zsolt-szabo-bln/0)

### Video Demonstration

<video src="https://github.com/Ainaganiu/Custom-Visual/raw/main/Budget%20Vs%20Actual%20KPI%20Card/svgZlots.mp4" controls width="600"></video>

## Native visual

Nothing to import. Card, bar, conditional formatting rule — all stock Power BI. Opens in Desktop, Service, mobile, no setup step for the reader.

Right-click still gives you Show data and Export data. Cross-filtering with the rest of the page just works, same as any built-in visual.

The trade-off is the shape. The badge and bar get close to Zsolt's original design but not exact — native visuals only bend so far before you're fighting the format pane. Good pick when people need to click into the numbers more than they need it to look identical to the reference.

## SVG visual (DAX)

One measure returns SVG markup, rendered through an image/HTML visual set to the Image URL data category. This is the version that matches the original almost exactly — the bar, the badge, the spacing, all of it.

What you give up for that: it behaves like an image, not a Power BI object. No Show data, no export, no cross-filter unless you wire click behavior in yourself. SVG rendering on mobile is inconsistent too — test it there before shipping, don't assume it looks the same as Desktop.

It still updates live off filters and slicers, since the SVG regenerates from the DAX each time. Works well for exec decks and PDF-style reports, where people are reading the number, not digging into it.

## PBIVIZ visual

Custom-coded, packaged as a `.pbiviz` file. Shows up in the visualizations pane with a real Format pane — colors, thresholds, labels, whatever got exposed in code.

Has to be imported once before anyone can use it — from a file, an org visual store, or AppSource once certified. In locked-down tenants that might need admin approval first. After that one-time step, it behaves the same everywhere: Desktop, Service, mobile.

This is the only build where tooltips, selection, and cross-filter are a decision made in code, not something you get by default. Whatever wasn't built in doesn't exist — no quick format-pane fix, that needs a dev cycle.

Makes the most sense when the same visual is going into a bunch of reports, or out the door to other people, and it needs to behave the same everywhere it lands.

## At a glance

| | Native | SVG | PBIVIZ |
|---|---|---|---|
| Setup before use | None | None | Import once |
| Matches the design | Close | Closest | Closest |
| Data interaction (filter/export) | Yes | No, unless built in | Yes, if built in |
| Mobile | Fine | Inconsistent | Fine |
| Customizing later | Format pane | Edit the DAX | Format pane, if exposed |
