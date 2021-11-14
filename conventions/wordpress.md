# WordPress Conventions

## Version Tracking

- Only themes are to be committed, starting from the theme root directory. (leave off wp-content/themes/themeName structure)

- Ignore vendor folders, such as node_modules, vendor, etc.

- dist-folder has to be committed. (doesn't matter if it happens to end up to be a development build, just make sure to have SOMETHING workable)

## Plugin stack

- WordFence
- UpdraftPlus (or any other backup solution)
- Regenerate Thumbnails
- Better Search Replace (has to be removed on production sites)
- EWWW Image Optimizer (only optimizer that has decent webp support and works without cloud converting services)
- Advanced Custom Fields Pro
- Custom Posttypes UI (CPT)
