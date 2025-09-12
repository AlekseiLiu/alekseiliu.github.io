# Social Icon Overrides (saved for later)

These were experimental size adjustments applied via `_sass/_overrides.scss` and an `@import "overrides";` at the end of `assets/css/main.scss`.

```scss
// Minimal override: gently reduce social icon size when inside profile
.profile .social .contact-icons {
    font-size: 2.4rem; // was 4rem
}

// Adjust raster and svg images proportionally
.profile .social .contact-icons a img {
    width: 1.6rem; // was 3.2rem
    height: 1.6rem;
    margin-bottom: 0.7rem; // slightly tighter
}

.profile .social .contact-icons a svg,
.profile .social .contact-icons a svg image {
    width: 2rem; // was 3.5rem
    height: 2.4rem; // svg container height
}

// Slightly reduce spacing below the block when embedded near header
.profile .social {
    margin-bottom: 0.5rem;
}
```

Re-apply steps later:
1. Restore this file as `_sass/_overrides.scss`.
2. Ensure `@import "overrides";` is the last line in `assets/css/main.scss` after all other imports.
3. Hard refresh the browser.

Rollback steps (already done except for deletion of the original file):
1. Remove the import line from `main.scss` if present.
2. Delete `_sass/_overrides.scss` (we will do this now).

