# simplesamlphp-module-themeclean

A theme for SimpleSAMLphp 2.5: a modern, accessible look for the end-user pages (login, discovery, consent, logout, error) and the admin pages. Not built yet; this repository holds what is needed to build it.

- `docs/theme-spec.md`: what to build, constraints, verified facts and open questions.
- `docs/design-brief.md`: the brief the design was drawn from. It also covers the Console module; only the theme parts apply here.
- `design/`: export of the design boards and `console.css` (tokens and components).

Once built:

```php
'module.enable' => ['themeclean' => true],
'theme.use' => 'themeclean:clean',
```
