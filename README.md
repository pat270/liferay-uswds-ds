# USWDS Design System for Liferay

A Liferay workspace that packages the [U.S. Web Design System (USWDS)](https://designsystem.digital.gov/)
as a portable, drop-in bundle any DXP site can adopt. It ships as two client extensions:

1. `liferay-uswds-theme-css` (type: `themeCSS`)
2. `liferay-uswds-site-initializer` (type: `siteInitializer`)

## Contents

```
liferay-uswds-workspace/
├── settings.gradle                     ← workspace plugin (com.liferay.gradle.plugins.workspace 16.0.6)
├── gradle.properties                   ← liferay.workspace.product=dxp-2024.q2.8
├── package.json                        ← yarn workspaces
├── client-extensions/
│   ├── liferay-uswds-theme-css/
│   │   ├── client-extension.yaml       ← type: themeCSS (clay.css + main.css)
│   │   ├── package.json                ← @clayui/css, baseTheme: styled
│   │   └── src/
│   │       ├── frontend-token-definition.json   ← Seam 1: tokens (exposed in Style Books)
│   │       └── css/
│   │           ├── _clay_variables.scss         ← Seam 2: SCSS overrides BEFORE Clay compiles
│   │           ├── _custom.scss                 ← Seam 3: arbitrary CSS AFTER Clay
│   │           ├── components/ portlet/ custom_properties/
│   │           └── ...
│   └── liferay-uswds-site-initializer/
│       ├── client-extension.yaml       ← type: siteInitializer + oAuthApplicationHeadlessServer
│       └── site-initializer/           ← source of truth for markup — EDIT HERE
│           ├── thumbnail.png
│           ├── fragments/group/uswds/  ← collection + resources + fragments uswds-*
│           ├── layout-page-templates/master-pages/main/
│           ├── layouts/                ← 01_landing-page, 02_create-account, 03_sign-in, 04_form
│           ├── layout-set/public/metadata.json
│           └── site-navigation-menus.json
└── docs/
    └── poc-report.md                   ← full POC write-up
```

## What you get

- **Site initializer** that creates the **Liferay USWDS** site (ERC `liferayUswds`), with its
  fragments, master page, pages and Main Navigation menu.
- **Theme CSS client extension** `liferay-uswds-theme-css`: Clay compiled against USWDS values,
  with a 759-token `frontend-token-definition.json` that Style Books can edit.
- 8 fragments under the `uswds-*` namespace: `uswds-header`, `uswds-footer-medium`,
  `uswds-footer-slim`, `uswds-alert`, `uswds-landing-page`, `uswds-create-account`,
  `uswds-sign-in`, `uswds-form-page`.
- 4 pages: Landing Page, Create Account, Sign In, Form.

## Build and deploy

From the workspace root, deploy the theme first, then the site initializer:

```bash
./gradlew :client-extensions:liferay-uswds-theme-css:deploy
./gradlew :client-extensions:liferay-uswds-site-initializer:deploy
```

(Or run `gradlew deploy` inside each client extension directory.)

`settings.gradle` and `gradle.properties` point at `../../.m2-tmp` and `../../../bundles`, which
assume the workspace lives at `workspaces/liferay-uswds-workspace/` inside a `liferay-portal`
checkout. Change those paths if you run it somewhere else.

The site initializer's OAuth headless server is set to `localhost:8080` over `http`. Change
`.serviceAddress` / `.serviceScheme` in its `client-extension.yaml` for other hosts.

## Apply the theme (post-site creation)

The site is created with `themeName: "Classic"`. Apply the USWDS theme CSS manually:

1. Go to **Control Panel → Sites**, open the **Liferay USWDS** menu (⋮) and choose **Go to Pages**.
2. In the Liferay USWDS site, open **Product Menu → Site Builder → Pages**.
3. Click ⋮ in the Control Menu and choose **Configuration**.
4. Scroll to **Theme CSS Client Extension** under *Customization* and click **+**.
5. Select **Liferay USWDS Theme CSS** and click **Save**.
6. Click **Home** in the Product Menu to view the theme.

## Edit / iterate

There are three override seams in the theme. Use the highest one that works:

1. **Tokens:** `src/frontend-token-definition.json`. Style Books expose these.
2. **SCSS variables:** `src/css/_clay_variables.scss`, applied before Clay compiles.
3. **Custom CSS:** `src/css/_custom.scss`, applied after Clay. Use it only when no token or
   variable covers what you need.

Edit fragments, layouts, the master page and navigation under
`client-extensions/liferay-uswds-site-initializer/site-initializer/`.

## Best practices

- **Token-first, always.** Override variables, never rules. If you are writing a selector to
  override a compiled Clay rule, you probably want to override a token instead.
- **Use semantic tokens, not hue tokens.** Map USWDS hues onto `--primary`, `--success`, `--info`,
  etc., and let fragments reference those. `btn-primary` keeps working after a rebrand;
  `bg-green-cool-vivid-40` does not.
- **Never hardcode hex/rgb in fragment CSS or HTML.** The token surface is the contract.

For example, `primary` maps to a USWDS hue in `frontend-token-definition.json`:

```json
{
	"defaultValue": "var(--blue-vivid-60)",
	"label": "primary",
	"name": "primaryColor",
	"mappings": [{"type": "cssVariable", "value": "primary"}]
}
```

`uswds-alert` then uses `alert-primary` and never names a color, so changing `primary` in a Style
Book rebrands every alert, button and background at once.

## Fragment notes

- Each fragment is a directory with `fragment.json`, `index.html`, `index.css`, an optional
  `index.js` and `configuration.json`.
- Use `data-lfr-editable-*` attributes / `<lfr-editable>` so content authors can edit text,
  links and images in the Page Editor.
- `<lfr-widget-nav></lfr-widget-nav>` renders the site navigation and
  `<lfr-widget-search-bar></lfr-widget-search-bar>` renders search inside custom
  header/footer fragments.
- Page definitions reference fragments by key, e.g. `{"fragment": {"key": "uswds-alert"}}`.

## Compatibility

- Tested on Liferay CE 7.4.3.148 GA148.
- DXP 2024.Q2.x and newer quarterly releases should work, with possible minor token / Style Book
  drift between quarters.
- DXP 2024.Q1 likely works but is unverified; ColorPicker token editing is the risk.
- DXP 2023.Q4 has themeCSS, siteInitializer and OAuth headless server, but token editor types may
  not match.

## Known limitations

- The two client extensions have to be deployed separately.
- The theme CSS client extension has to be applied by hand (see above).
- The new site may not appear under *Control Panel → Sites* until you create another, unrelated
  site.
- `lfr-widget-nav` outputs old markup: the active class is on the `li` instead of the `nav-link`.
- It is easy to end up with too many fragments, which leads to clutter.

See [`docs/poc-report.md`](docs/poc-report.md) for the full write-up.
