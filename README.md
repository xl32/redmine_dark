# Redmine Dark Static

A standalone Redmine 7 theme derived from `fraoustin/redmine_dark`.

Unlike the original plugin, this package does not register a Redmine plugin,
inject CSS into every theme, add a cookie, or add a “dark mode” link. Dark mode
is active only when this theme is selected in **Administration → Settings →
Display → Theme**.

## Installation for Redmine 7

Copy the `redmine_dark_static` directory to Redmine's root `themes` directory:

```text
/usr/src/redmine/themes/redmine_dark_static
```

For Docker, persist it with a host mount such as:

```yaml
volumes:
  - /srv/redmine/themes:/usr/src/redmine/themes
```

Then recreate Redmine and select `redmine_dark_static` in the administration UI.

## Important

Remove or move the original `/usr/src/redmine/plugins/redmine_dark` plugin.
Keeping it installed will continue loading `dark.css` and `dark.js` into all
other themes, including the default blue theme.

## License

GNU GPL version 2, matching the source plugin. See `LICENSE`.
