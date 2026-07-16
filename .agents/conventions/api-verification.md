# API Verification (Hooks, Functions, Classes)

Never reference a hook, function, class, method, or constant you have not
verified exists. A name that looks plausible — or appears often in training
data — is not evidence. Hook names are plain strings, so no linter, PHPCS
rule, or CI check catches an invented one, and nothing warns you when a
plugin function does not exist on the target site.

## Failure modes this prevents

- `add_action()` / `add_filter()` registered on a hook name no code ever
  fires — the callback silently never runs, and the change ships "green".
- Calling a plugin function that does not exist on the site — fatal error
  in production.
- A callback signature that does not match the call site (wrong argument
  count or order, missing `$accepted_args`) — `ArgumentCountError` or
  silently missing data.
- Using an API added, changed, or removed in a plugin version different
  from the one the site actually runs.

## Verification ladder

Work down the ladder; stop at the first level that gives a definitive
answer.

### 1. The repository source

Search the codebase for the definition or the hook's call site:

```bash
# A function or class:
grep -rn "function wc_get_orders" wp-content/
grep -rn "class WC_Order" wp-content/

# A hook — find where it is actually fired:
grep -rn "do_action( 'woocommerce_checkout_order_processed'" wp-content/
grep -rn "apply_filters( 'woocommerce_checkout_fields'" wp-content/
```

Note that third-party plugins are often not committed to Team51 site
repositories — an empty grep result means "not in the repo", not "does not
exist". Move down the ladder before concluding either way.

### 2. Official source or documentation

When the source is not in the repository:

- **WordPress core** — look the API up on
  `https://developer.wordpress.org/reference/` (covers functions, hooks,
  classes, and methods, with signatures and since-versions).
- **Plugins** — read the plugin's actual source
  (`https://plugins.svn.wordpress.org/{slug}/`, or the plugin's GitHub
  repository) or its official developer documentation. Verify against the
  version the site runs, not the latest.

### 3. The staging clone (implement runs)

When you have WP-CLI access to a staging clone (never production):

```bash
# Is the plugin actually active, and which version?
wp plugin list --status=active

# Locate the installed plugin source, then grep it:
wp plugin path <slug> --dir

# Direct existence check on the running site:
wp eval 'var_dump( function_exists( "wc_get_orders" ) );'
```

### 4. Cannot verify — do not use it

If none of the above confirms the API exists:

- Prefer a WordPress core API you **can** verify.
- If a cross-plugin call is unavoidable, guard it with
  `function_exists()` / `class_exists()` **and** call it out explicitly in
  the PR description so a human reviews it.
- Never ship an unverified hook registration — it fails silently.

## Hook rules

- **Copy hook names verbatim from the `do_action()` / `apply_filters()`
  call site.** Never type a hook name from memory.
- **Match the callback to the call site.** Count the arguments actually
  passed and register with an explicit `$accepted_args`:

  ```php
  // Call site, verified in the plugin source:
  // do_action( 'a8csp_events/registered', $event_id, $user_id );

  add_action( 'a8csp_events/registered', 'a8csp_notify_organizer', 10, 2 );
  ```

- **Filters must return a value.** Confirm the type of the first argument
  from the call site and return the same type.
- **Confirm timing and context.** Some hooks fire only in admin, only on
  the front end, or not at all during REST or WP-CLI requests. If the
  change depends on when a hook fires, verify where the call site lives.

## Plugin function rules

- **Active, not just installed.** A plugin present in the repo or on disk
  may be deactivated on the target site — confirm with
  `wp plugin list --status=active` (staging clone) or with the site team.
- **Guard cross-plugin calls even after verifying**, so a deactivated
  plugin degrades gracefully instead of fataling:

  ```php
  if ( function_exists( 'wc_get_orders' ) ) {
      $orders = wc_get_orders( array( 'limit' => 10 ) );
  }
  ```

- **Check version-sensitive APIs.** If the API was added or changed
  recently, confirm the installed version supports it before using it.
- **Well-known families are where hallucinations look most plausible.**
  `wc_*` (WooCommerce), `get_field()`/ACF, `gform_*` (Gravity Forms) are
  real API families, but every individual name and signature still needs
  verification — a convincing-looking sibling function may not exist.

## Existence is not behavior

An API can exist, be documented, and be valid PHP — and still be
silently ignored in the context the change targets. Verify that the
mechanism actually takes effect where you are using it, not just that
the name resolves.

Known trap (real incident): a taxonomy registered with a custom
`meta_box_cb` to render radio buttons instead of checkboxes. The
argument exists and the callback ran through every linter — but the
block editor never renders taxonomy meta boxes (core registers them
all with `__back_compat_meta_box => true`, which the block editor
skips), so editors saw the default checkbox panel and the radio UI was
unreachable. Custom taxonomy UI **is** achievable — via the
`editor.PostTaxonomyType` JavaScript filter, not PHP alone; see the
`block-editor-development` skill.

The general rule: classic-editor-era mechanisms (taxonomy/meta-box
callbacks, edit-screen hooks, admin notices on the editor screen) must
be confirmed to apply in the block editor before relying on them. And
never assert in a code comment, docblock, or PR description that a UI
behaves a certain way unless that behavior was observed in the target
context or confirmed in the platform source — a wrong behavior claim
misleads every human and AI reviewer downstream.

## Static analysis is only a backstop

PHPStan flags undefined functions, classes, and methods when the project
runs it — but it cannot validate hook names (plain strings) or
`$accepted_args` mismatches, and many theme repositories do not run
PHPStan at all. Do not rely on CI to catch an invented API.

## Checklist

Before shipping code that references an API this change did not define:

- [ ] Found the definition or hook call site (repo, official source, or
      staging clone) — not recalled from memory.
- [ ] Hook names copied verbatim from the call site.
- [ ] Callback signatures and `$accepted_args` match the call site.
- [ ] Cross-plugin calls guarded with `function_exists()` /
      `class_exists()`.
- [ ] The version on the target site supports the API.
- [ ] The API takes effect in the target context (block editor vs
      classic editor, admin vs front end, REST vs web) — not just
      exists.
- [ ] Anything unverifiable is flagged in the PR description.
