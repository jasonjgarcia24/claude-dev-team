# Surface: browser app

Probing tool: Chrome DevTools MCP (via the `browser-testing-with-devtools` skill).

## How to probe

- Open the target via `new_page` / `navigate_page`
- Drive interactions via `click`, `fill`, `fill_form`, `type_text`, `press_key`, `hover`, `drag`
- Inspect DOM via `take_snapshot`; capture visuals via `take_screenshot`
- Read runtime signals via `list_console_messages`, `list_network_requests`, `get_console_message`, `get_network_request`
- Emulate conditions via `emulate` (network throttle, device, offline), `resize_page`
- For performance traces: `performance_start_trace` / `performance_stop_trace` / `performance_analyze_insight`
- For Lighthouse + a11y: `lighthouse_audit`

## Evidence to capture

- Screenshot at each decisive moment (pass a filePath into `take_screenshot` to save)
- Relevant console messages — quote key lines in the report
- Failed network requests — URL, method, status, timing

## Stage-specific additions

### MVP
- Render at mobile viewport (375px wide) — use `emulate` or `resize_page`
- No console errors on the primary flow

### Beta
- Keyboard-only navigation: tab through the flow, focus visible, tab order logical
- Slow network: emulate Slow 3G via `emulate` — flow must still complete
- Viewport coverage: 375px / 768px / 1440px

### GA
- Performance: `lighthouse_audit` — LCP < 2.5s, CLS < 0.1 on the primary flow
- Accessibility: axe-equivalent audit via `lighthouse_audit`, contrast pass, form labels associated
- Degraded network / offline: app must render a useful state, not white-screen
- No layout shifts on async content load
