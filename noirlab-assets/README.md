# NOIRLab Style Identity Assets

- `source/noirlab_logo.svg` is the official NOIRLab vector logo.
- `source/aura_logo.svg` is the official AURA vector logo.

The SVG files are the maintained identity sources. The vector PDF files beside this README are the runtime assets loaded by `noirlab-document.sty`, and are the only files LaTeX reads. They are committed, so using the style does not require an SVG converter.

Regenerate the PDF derivatives from the SVG originals with any converter that preserves vectors, for example:

```sh
rsvg-convert --format=pdf1.5 --keep-aspect-ratio \
  --output=noirlab_logo.pdf source/noirlab_logo.svg
```

Record the official download URLs and applicable identity guidance before a controlled public release.
