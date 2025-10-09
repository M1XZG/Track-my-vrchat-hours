# Templates

This folder holds the README template and the insertion snippet used by the automation.

Files:

- `README-template.md` — Your README template. Create this by copying your current `README.md` and then placing the snippet where you want the playtime line to appear.
- `README-template-snippet.md` — A small block you can paste/append into your template. It contains the `myhoursHERE` insertion markers.

Setup (macOS zsh):

```zsh
mkdir -p templates
cp README.md templates/README-template.md
cat templates/README-template-snippet.md >> templates/README-template.md
```

Then open `templates/README-template.md` and move the block to the exact location you want in your README.

Markers must remain exactly as:

<!-- start myhoursHERE -->
<!-- end myhoursHERE -->
