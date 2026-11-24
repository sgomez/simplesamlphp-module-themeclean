# simplesamlphp-module-themeclean

A theme for SimpleSAMLphp 2.5: a modern, accessible look for the end-user pages (login, discovery, consent, logout, error) and the admin pages. Not built yet; this repository holds what is needed to build it.

- `docs/theme-spec.md`: what to build, constraints, verified facts and open questions.
- Branch `prototype/theme` (throwaway, never merged): the design boards, `console.css` and the brief they were drawn from.

Once built:

```php
'module.enable' => ['themeclean' => true],
'theme.use' => 'themeclean:clean',
```
