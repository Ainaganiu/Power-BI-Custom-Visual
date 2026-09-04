# Power BI Custom Visuals

A curated collection of custom visuals, templates, and helper files for Power BI Desktop and Power BI Service. This repository gathers different visual types (HTML, SVG, Deneb templates, and .pbiviz packages) to make it easier to test, share, and re-use custom visuals in your reports.

## What’s in this repository

- HTML visuals — wrappers and examples that render HTML/CSS/JS inside Power BI visuals.
- SVG visuals — lightweight, resolution-independent visual components created with SVG.
- Deneb templates — Vega / Vega-Lite templates and examples for the Deneb custom visual.
- .pbiviz packages — packaged Power BI custom visuals ready to import into Power BI Desktop.

## How to use

1. Browse the folders to find the visual or template you want to try.
2. If the folder contains a `.pbiviz` file, open Power BI Desktop and use `External tools` → `Import from file` (or `Visualizations` pane → `...` → `Import from file`) to add the visual to your report.
3. For Deneb templates, open Deneb in Power BI and paste the Vega / Vega-Lite JSON or import the template file.
4. For HTML/SVG examples, review the included README or example files in the folder for usage notes and any required configuration.

Security note: Custom visuals that use HTML/JS can execute scripts and access remote resources. Only use visuals you trust and review the source before importing them into production reports.

## Contributing

Contributions, improvements, and new examples are welcome.

- Fork the repository and create a branch for your changes.
- Add or update visuals / templates and include a short README describing how to use them.
- Open a pull request describing the change.

If you plan to publish a visual to AppSource or distribute it widely, include build, inform me and packaging instructions in the visual's folder.

## License
MIT

## Contact

Created by @Ainaganiu — open an issue or submit a pull request if you have suggestions or found problems.
