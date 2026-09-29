# QUC 2026 slides

[Slidev](https://sli.dev/) presentations for the QGIS User Conference 2026:

- [Macro profiler](./macro-profiler.md) talk
- [QGIS plugin dev workshop](./qgis-plugin-dev-workshop.md)

## Usage

```sh
npm install
npm run dev:macro-profiler     # for macro-profiler slides
npm run dev:workshop           # for plugin dev workshop slides
```

Every push to `main` builds both decks and their PDFs and deploys them to GitHub Pages at <https://cofactor-company.github.io/quc2026-slides/> (see [`.github/workflows/pages.yml`](./.github/workflows/pages.yml)).

## License

- Slide content, images and notes: [CC BY 4.0](./LICENSE)
- Code (`components/`, `slide-bottom.vue`): [MIT](./LICENSE-CODE)
