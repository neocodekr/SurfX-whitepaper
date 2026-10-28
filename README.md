# SurfX Whitepaper

Vue whitepaper reader for SurfX. The main website is at [www.surfx-protocol.com](https://www.surfx-protocol.com/).

The reader presents all 25 chapters and 14 tables from `public/SurfX_Whitepaper_KO_v0_2.docx` in Korean. The English view provides chapter titles and explanatory summaries alongside the complete Korean source text; it is not a full English translation. The structured source is in `src/whitepaper.ko.json`, and navigation labels and English summaries are in `src/whitepaperMeta.js`.

```sh
npm ci
npm run dev
npm run build
```

GitHub Actions deploys the Vite build to GitHub Pages at `https://whitepaper.surfx-protocol.com/`.
