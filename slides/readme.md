# Generating Slides

I'm using [Marp](https://marp.app) CLI to generate slides from markdown files.

If you want to generate the slides, you can use the following command:

```bash
marp slides/slides.md --output slides/slides.html
```

Or via npx (npm exec) without installing Marp CLI globally:

```bash
npx @marp-team/marp-cli slides/slides.md --output slides/slides.html
```

In addition the VS Code extension for Marp can be used to preview the slides in the editor.