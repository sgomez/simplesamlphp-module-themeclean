# Theme spec

## Goal

A SimpleSAMLphp 2.5 theme, module `themeclean`, theme `clean`, that replaces the stock look on every page a person or an administrator sees. It shares one design system with the Console module (`sgomez/simplesamlphp-module-console`), which is a separate package and not a dependency in either direction.

## What to build

Two shells, both drawn in `design/`:

- **Shell A**, centred card with no navigation, for end-user pages. Boards: `Login`, `LoginMobile`, `LoginMulti`, `LoginMultiMobile`, `Discovery`, `DiscoveryMobile`, `Consent`, `Logout`, `Error`, `ErrorMobile`.
- **Shell B**, sidebar application shell, for admin pages. Boards: `Sidebar` (modes `core` and `full`), `Configuration`, `TestAuth`, `Federation`, `MobileList`, `MobileNav`. `Main` shows the shell with the Console's entries, for reference only.

`design/console.css` is plain CSS and is the starting stylesheet: tokens for light and dark, and the component classes. `DesignSystem` documents them.

Each `*.dc.html` is a board in the design tool's component format, not a standalone page: read the markup inside `<x-dc>` and the data in the script block. `canvas.json` lists board titles; it also names Console boards that are not copied here.

## Constraints

- Twig templates only, overriding core's. No build step, no bundler, no framework: one hand-written stylesheet and at most one small hand-written script.
- Core's default CSP stays: no inline styles, no inline scripts. The boards carry inline `style` attributes; that is the design tool's output and must become classes.
- Fonts (Inter, JetBrains Mono) are served from the module. No request to Google Fonts or any third party.
- `.ui.dark` in `console.css` becomes `@media (prefers-color-scheme: dark)`.
- Every end-user page works without JavaScript. Keyboard access, visible focus, WCAG 2.2 AA contrast, logical properties for RTL.
- Languages are SimpleSAMLphp's: the language selector lists what core reports as available and English is the fallback. The theme adds no language handling of its own and translates only strings it introduces.
- Pages of modules the theme does not override must still render acceptably inside the new `base.twig`.
- The admin sidebar is built from core's admin menu, so entries added by other modules (`cron`, `metarefresh`, the Console) appear without the theme knowing them.

## Facts from the SimpleSAMLphp 2.5 documentation

Sources: `simplesamlphp-theming` and `simplesamlphp-modules` in the 2.5 docs.

- A theme is a module. Core templates are overridden in `themes/<theme>/default/`, another module's in `themes/<theme>/<module>/<template>`; anything not overridden falls back to the original.
- Enabled with `'module.enable' => ['themeclean' => true]` and `'theme.use' => 'themeclean:clean'`.
- The docs name `base.twig`, `_header.twig` and `_head.twig` as the usual overrides.
- Static files go in `public/assets/` and are referenced with `{{ asset('style.css', 'themeclean') }}`.
- A theme may set `theme.controller` to a class implementing `\SimpleSAML\XHTML\TemplateControllerInterface` (`setUpTwig`, `display`) to add Twig functions or data.
- Inline `<style>` is blocked by the default CSP; the docs advise keeping the CSP and using a linked file.

## Not verified: read core's source before relying on it

- `composer.json` here (type `simplesamlphp-module`, `composer-module-installer`) is written from memory of how official modules are packaged.
- Which variables core passes to templates for the language selector, the current language and RTL.
- How the admin menu is exposed to templates, and whether the theme can render it as a sidebar without a controller.
- The template names and variables of each page to override: `core` (login, logout, error, no cookie), `saml` (discovery), `consent`, `multiauth`, `admin`.
- Whether `asset()` adds cache busting.
- Which block names third-party module templates expect from `base.twig`.

## Suggested order

1. Install SimpleSAMLphp 2.5 locally with `exampleauth`, `multiauth`, `consent` and `admin` enabled, and settle the unverified list against its source.
2. `base.twig` for both shells, the stylesheet and the fonts.
3. Login, then error, logout, discovery, consent, multiauth.
4. Admin pages and the sidebar.
5. Check every page in light, dark, a narrow viewport, an RTL language and with JavaScript off.
