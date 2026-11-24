# Design brief: a new interface for SimpleSAMLphp and its Console

## What to design

One design system and one application shell that serve two things:

1. **A theme for SimpleSAMLphp 2.5** that replaces its stock look: the pages end users see (login, IdP discovery, consent, logout, errors) and the administrator pages core ships (Configuration, Test, Federation).
2. **The Console**, a new administration module that manages SAML entities and authentication sources in a database instead of hand-edited PHP files.

The Console is designed first and in full. The theme is designed far enough to prove the shell, the navigation and the components hold for pages we do not own. Nothing here should need redesigning when the theme is built later.

We are not an official SimpleSAMLphp project. Do not reuse its visual identity (red header, dark tab strip, PureCSS look). Design something current, calm and dense enough for daily administrative work.

## The bar to clear

The Console replaces editing a PHP array in a text editor. An administrator who knows that file must find the Console **faster and less error-prone, never more work**. Every screen is judged against that: if doing it in the file is easier, the screen is wrong.

What the file does well, and the interface must keep:

- You only see the options you set. A 64-option authsource is ten lines if you use ten.
- You can read the whole object at a glance and copy it.
- Search is instant (Ctrl+F).

What the file does badly, and the interface must win on:

- No validation until production breaks.
- No idea what options exist or what they accept.
- No history, no undo, no review before it goes live.
- Secrets sit in clear text next to everything else.
- Every node needs the file copied to it.

## Users

- **The administrator** (primary, Console and admin pages). Technical, knows SAML terms, works on a desktop, often with the SP's or IdP's documentation open in another window. Uses the Console a few times a week, not all day: they forget where things are between visits. May manage 5 entities or 500.
- **The end user** (theme only). Anyone logging in: any age, any device, often a phone, possibly a screen reader, possibly not English-speaking. Sees the login page for seconds and wants to leave it.

## Technical constraints

These are fixed. A design that breaks one cannot be built.

- Server-rendered **Twig** templates. No SPA, no client-side framework, no bundler, no build step.
- **One static, hand-written JavaScript file**, no dependencies. It enhances markup through `data-` attributes; it validates nothing and knows no schema. Pages must be usable the moment HTML arrives.
- **Content Security Policy `script-src 'self'; style-src 'self'`**: no inline `<script>`, no inline `style=""` attributes, no inline event handlers, nothing from a CDN.
- **No external fonts or assets.** Use a system font stack, or fonts that can be self-hosted under an open licence. Icons are inline SVG or a self-hosted sprite.
- Forms are **generated from JSON Schema**. The form design must be a small set of field types and layout rules that a generator can apply to any schema, not hand-arranged pages. Presentation hints available per option: title, description, group, order, widget (`textarea`, `radio`), a "sensitive" flag, and "only applies when a sibling option has one of these values".
- Every state change is a normal form POST followed by a redirect. Design for full page loads: no optimistic updates, no toasts that depend on staying on the page. Feedback after an action is a message on the next page.
- **Light and dark themes**, following the system setting, through CSS custom properties.
- **Accessibility: WCAG 2.2 AA.** Keyboard reachable everywhere, visible focus, no meaning carried by colour alone, 4.5:1 text contrast, targets at least 24×24 px, honours `prefers-reduced-motion`.
- **Responsive.** End-user pages are mobile first. Admin and Console pages are desktop first but must remain usable at tablet width, with the navigation collapsing.
- **Languages.** The Console is English only for now. End-user and core admin pages are translated into many languages, some right-to-left, with strings that can be twice as long as English. Layouts must not depend on English string lengths; use logical properties so RTL works.

## The shell: one layout system, two shells

The menu layout is the main thing to get right, because the theme and the Console share it.

### Shell A: centred, no navigation (end-user pages)

For login, IdP discovery, consent, logout and error pages.

- A single centred column, one task per page, no navigation.
- Slots: organisation logo and name (set by the deployer), the page's task, a language picker, a quiet footer (help link, privacy link, deployer text).
- The deployer must be able to rebrand it by changing a logo, an accent colour and a few strings. Design the tokens so one accent colour re-skins it safely in both themes.

### Shell B: sidebar application (administrator pages)

For core's admin pages and for the Console, as one continuous application.

- **Persistent left sidebar** with grouped sections, a **top bar** with breadcrumbs on the left and the global action on the right, and a content area with a constrained reading width.
- **The sidebar's items are not fixed.** Core contributes its own (Configuration, Test, Federation), and any installed module can add entries. Design the sidebar for an open list: groups with headings, items with an optional count or status badge, one level of nesting at most, graceful behaviour with 4 items and with 25. A module's section must look native without that module shipping any CSS.
- Proposed grouping, to refine:
  - **Overview**
  - **Entities**: Service providers, Identity providers, Hosted IdP, External entities
  - **Login**: Authsources
  - **Operate**: Publish, History, Nodes
  - **System** (core's pages): Configuration, Test authentication, Federation
  - Foot: signed-in identity, log out, language.
- The top-bar global action belongs to the Console ("Publish 7 changes"). On core pages with no Console installed the slot is empty; the shell must look complete without it.
- Below roughly 900 px the sidebar collapses to a button that opens it as an overlay.
- Core's admin pages are mostly read-only reports (module list, PHP checks, metadata dumps, certificate downloads, an attribute table after a test login). Show at least one of them in Shell B to prove the components cover it.

## Design system to deliver

- **Tokens**: colour (neutral scale, one accent, success, warning, danger, info; each with a text, a background and a border value, in light and dark), type scale, spacing scale, radii, elevation, focus ring.
- **Typography**: a UI sans and a monospace. Entity IDs, authsource ids, option names, URLs, file paths and PHP are always monospace. Entity IDs are long URLs: define how they truncate (middle ellipsis, full value on hover and on copy) and how they wrap.
- **Components**, each with hover, focus, disabled, error and loading states where they apply:
  - Buttons: primary, secondary, quiet, destructive; sizes; with icon.
  - Inputs: text, number, select, textarea, search, switch, checkbox, radio group, secret field with reveal, copy-to-clipboard field.
  - List editor: a repeatable row of one or several controls with add, remove and reorder. Used constantly (endpoints, attributes, certificates, language maps, contacts).
  - Language map editor: rows of language and text.
  - Certificate field: pasted PEM that collapses into a summary (subject, expiry, fingerprint) once valid.
  - Key reference field: a file name or path on each node, with a "found / not found on this node" indicator.
  - Status indicator (dot plus text) and label chip. Keep the two visually distinct: status is where the object stands in the publish cycle; a chip is a fact about it.
  - Callout: neutral, info, warning, error; with an optional action.
  - Object list row: title, monospace subtitle, chips, status, affordance to open.
  - Data table for reports (sortable header, sticky header, zebra optional).
  - Diff: unified, monospace, of PHP code, with syntax colouring kept inside added and removed lines, collapsed unchanged regions, and "secret changed" as its own treatment.
  - Code block with syntax colouring for PHP and XML, and a copy button.
  - Tabs, segmented control, breadcrumbs, pagination, dropdown menu, modal confirmation, empty state, skeleton/loading state, page-level success and error messages.

## Console: concepts the design must make obvious

Use these words exactly; they are the product's vocabulary.

- **Managed entity**: a SAML entity edited in the Console. Three kinds ("sets"), shown with a friendly name and the technical one beside it in small monospace:
  - Service providers (`saml20-sp-remote`)
  - Identity providers (`saml20-idp-remote`)
  - Hosted IdP (`saml20-idp-hosted`)
- **External entity**: an entity that reaches SimpleSAMLphp from files or federation feeds. Shown, never edited. There may be thousands.
- **Managed authsource**: an authentication source edited in the Console. Five types: `saml:SP`, `ldap:Ldap`, `sqlauth:SQL2`, `multiauth:MultiAuth`, `exampleauth:UserPass`.
- **File authsource**: one declared by hand in `authsources.php`. Listed read-only.
- **Saving is not publishing.** Edits are saved to the database and go nowhere. A **Publication** validates everything together and produces a numbered **Snapshot**, which every **Node** then fetches. This two-step model is the most important thing for the interface to teach, and the most likely to confuse. The number of unpublished changes and the way to publish them must be visible from every screen.
- **Pending change**: a saved difference with the live Snapshot. An object is: Published, Modified, Not published yet, To be withdrawn (disabled), or Deleted.
- **Rollback**: making an earlier Snapshot live again. It does not touch what is saved.
- **Override**: a managed entity that replaces an External entity with the same entity ID. **Shadowed**: the opposite, the external one wins and the managed one is ignored on that node.
- **Import**: creating managed entities from External ones, one or many.
- **Secret reference**: a placeholder like `%secret(name)%` or `%env(CONSOLE_NAME)%` stored instead of a real secret.
- **Audit trail**: the append-only record of every change.

## Console screens

For each: what it is for, what it shows, and the states to design.

### Overview

The landing page. Answers "is anything waiting for me?" in five seconds.

- A "needs attention" list, most urgent first, each line a link to the place that fixes it: unpublished changes, a blocked Publication, certificates expiring within 30 days, Overrides whose source changed, Shadowed objects, nodes that are behind or report problems, a pending database migration.
- Counts of entities and authsources, the live Snapshot and when it was published.
- States: first run (nothing managed yet: lead with "Add your first service provider" and "Import what you already have"), all clear, things to do.

### Entity list (one per set)

- Rows: name, entity ID, chips (warnings, Override, Shadowed, Source changed, Disabled), status.
- Search by name or entity ID, filter by status and by "has warnings". Must work from 3 to several hundred rows: design pagination and the filtered-empty state.
- Primary action: add one.
- The whole row opens the entity.

### Add an entity

- The main path is **paste the metadata XML** (or give its URL) and get a pre-filled form to review. Starting empty is the secondary path. Copying from an External entity is the third, cross-linked.
- Show what was understood from the pasted XML before saving, and what was ignored.

### Edit an entity / edit an authsource (the form)

The hardest screen. The largest schema, `saml:SP`, has about 64 options in 10 groups; most objects set 8 to 15.

- **Show only what is set**, plus required options. Each group states how many of its options are set and offers the rest on demand. A group with nothing set is one collapsed line.
- **Find an option** by its name or its title across the whole schema, including unset ones; choosing one adds it to the form and focuses it. This is the equivalent of knowing the option name when editing the file, and should feel that fast. Consider a keyboard shortcut.
- Each option shows: a human title, its real option name in monospace (administrators read documentation that uses the real name), a one-line description, the control, and its default when unset.
- Removing an option (returning it to "not set") must be as easy as adding it. Distinguish "not set" from "set to empty" and from "set to false".
- Conditional options appear and disappear as the option they depend on changes.
- A **"PHP" view** of the same object: read-only, exactly the array nodes will load, secrets hidden. It builds trust and helps people compare with their old files. Consider a side-by-side layout on wide screens.
- Validation: errors beside the field, a summary at the top that links to each one, an error count on each group and in the in-page index. Warnings (expired certificate, SHA-1, plain http endpoint) look different from errors and never block.
- Sensitive values: a password-style field with a reveal control; a Secret reference is shown as plain text and recognisably different from a literal.
- A persistent save area that states where the object stands: unsaved edits, saved but not published, published. Secondary actions (disable, discard changes, delete) sit together and away from Save.
- Leaving with unsaved edits warns.
- Contextual notices at the top when they apply: this is an Override (link to compare), its source changed, it is Shadowed, it is disabled.
- An in-page index of groups for long forms.
- States: new, clean, with unsaved edits, with errors after a rejected save, saved successfully, rejected because someone else changed it meanwhile.

### Authsource list

- Managed authsources as a list like entities.
- File authsources below, visibly read-only, with the note that the file wins when ids collide.
- Adding one starts by choosing among the five types; make that choice informative (one line on what each is for).

### External entities

- Grouped by source (type and location) and set. Read-only, searchable, paginated server-side; one source can hold thousands.
- Some sources allow bulk selection for Import, with a "select the whole source" control; others only allow copying one at a time. Entities already managed are marked and cannot be selected. Expired entities are marked, not hidden.
- Detail view of one External entity: its metadata as SimpleSAMLphp loads it, and "Create managed copy" with a plain statement of the consequence.
- Import flow: selection, a short confirmation, and a result page listing what was imported, what was skipped and why, what was converted, and which originals the administrator may now remove by hand.

### Compare an Override with its source

- The managed entity against the current External entity, option by option.
- One action: "Mark as reviewed", which silences the notice until the source changes again.

### Publish

The review-before-it-goes-live screen. Model it on a good pull request or deploy confirmation.

- Every Pending change as a row: added, modified, withdrawn or deleted; what kind of object; a one-line summary. Each expands to a **diff of the PHP**: the array nodes load now against the array they will load, as a unified diff with syntax colouring, the way an administrator would review a change to the file. Sensitive values never appear; a changed secret shows as a changed line with the value replaced by a marker. Secret references are shown as written, since each node resolves them itself.
- Blocking problems first, each with a link to the place that fixes it. Warnings collapsed below.
- A summary panel: the Snapshot number it will become, how many nodes it reaches, an optional comment, the Publish button, and why it is disabled when it is.
- Per-change "discard" and a global "discard all", both confirmed.
- States: nothing to publish, ready, blocked, the set of changes moved since the page was loaded (show it again), just published (success, with how nodes are picking it up).

### History

Two views of the past, under one entry.

- **Snapshots**: the retained Snapshots (10 by default) as a timeline: number, when, who, comment, number of changes, which is live. Any other one offers "Roll back to this".
- **Rollback confirmation**: what changes on the nodes, as a diff, and the statement that saved work is untouched.
- **Audit trail**: a feed of "who did what to which object, when", filterable by object, action and date. Each record opens to a diff and the full document after the change. A notice explains when every change is attributed to the constant identity `admin`.

### Nodes

- Each node: identifier, which Snapshot it serves, when it last checked, whether it reports problems. Rows that are behind, silent or in trouble stand out.
- Problem detail is only known for the node serving the page; say so, and show it: objects left out and why, a missing configuration fragment.
- States: single node (the common small deployment: keep it from looking empty or alarming), several healthy, some behind.

### Database migration notice

- After a module upgrade the database may need migrating before anything else works. A full-page, calm, single-action state: what is needed, that nodes keep serving meanwhile, one button.

## Theme screens (to the level of proving the system)

- **Login** (username and password): logo, organisation name, the two fields, submit, error state (wrong credentials), optional "remember username", optional organisation selector, help link. The name of the service being logged into, when known.
- **IdP discovery**: choose your organisation from a list that may hold thousands: search first, recently used on top, logos optional.
- **Consent**: which service is asking, which attributes will be released with their values, accept or decline, remember the choice.
- **Logout**: confirmation, and the partial-logout case where some services could not be logged out.
- **Error**: a human explanation, what to do next, a reference id to give to support, technical detail collapsed.
- **Admin, Configuration**: installed modules, checks with pass/warn/fail, links to tools.
- **Admin, Test authentication**: pick an authsource, then a table of the attributes received.
- **Admin, Federation**: hosted and remote entities as core lists them, metadata display with copy and download, certificate download, an XML-to-PHP metadata converter (a large textarea in, a code block out).

## Usability requirements

Apply these everywhere; call out in the design where each is satisfied.

- **Visibility of system status.** The administrator always knows: what is live, what is saved but not live, and whether the thing on screen has unsaved edits. Three states, never blurred.
- **Recognition over recall.** Option names, allowed values and defaults are shown, not remembered. Selects for closed sets, never free text.
- **Error prevention before error messages.** Constrain inputs, validate formats as early as a server-rendered page allows, confirm only what is destructive or hard to reverse (delete, discard, publish, rollback), and say in the confirmation exactly what will happen.
- **Good error messages.** Say what is wrong, where, and how to fix it, in plain words, next to the field. Never a bare "invalid value".
- **Undo and escape hatches.** Discard per object, discard all, rollback. Make them findable so people are not afraid to try things.
- **Progressive disclosure.** Common things visible, rare things one step away, nothing hidden without a hint that it exists.
- **Consistency.** One list pattern, one form pattern, one diff pattern, one place for the primary action on every page.
- **Speed for experts.** Keyboard access to navigation and to "find an option"; search on every list; links that can be bookmarked and shared for every object and every filtered view.
- **Clear hierarchy.** One primary action per page. Destructive actions never sit next to it or share its style.
- **Empty and first-run states** that explain what the area is for and offer the first step.
- **Honest loading and long lists.** Server-side pagination with a visible total; no infinite scroll.
- **Copy-friendliness.** Entity IDs, URLs, metadata and PHP can be copied in one click and selected without picking up decoration.
- **Plain language.** Short labels, sentence case, no jargon beyond the product's own vocabulary, and that vocabulary explained once where it first appears (a short definition on hover or beneath the heading).
- **Density with air.** Administrators scan lists; forms need room. Offer comfortable defaults, not a consumer-app sparseness.

## What we tried and what it taught us

A first prototype put the Console inside SimpleSAMLphp's own chrome with tabs and a form that printed all 64 options at once. It was rejected: it looked dated and was more work than the PHP file. A second prototype (light sidebar, top-bar Publish button, "only what is set" forms with an option finder, a PHP tab, paste-metadata entry, pull-request-style Publish page) went in the right direction; treat it as a floor, not a target.

## Deliverables

1. The design system: tokens (light and dark) and the component set above, with states.
2. Shell A and Shell B at desktop and mobile widths, including the sidebar with only core's items and with a full Console plus two extra module entries.
3. High-fidelity designs for every Console screen above, with the listed states. The form screen in three sizes of object: a small one (`multiauth:MultiAuth`, two options), a typical service provider, and `saml:SP` at full size.
4. The theme screens, at least Login, Discovery, Error and one core admin page, in both shells, to prove the system.
5. Notes on which behaviours need the JavaScript file and what the page looks like before it runs.
6. Anything in this brief you would change, and why: the grouping of the navigation, the screen list and the vocabulary shown to users are all open to challenge.
