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

The site is deployed with [Netlify](https://www.netlify.com/) (`netlify.toml`), and every pull request gets a deploy preview.

## License

- Slide content, images and notes: [CC BY 4.0](./LICENSE)
- Code (`components/`, `slide-bottom.vue`): [MIT](./LICENSE-CODE)
