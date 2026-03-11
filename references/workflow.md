# Workflow

## BrowserOS Execution Recipe

Use this concrete BrowserOS sequence before improvising:

1. Open Scholar Labs with `new_page`:
   - `https://scholar.google.com/scholar_labs/search?hl=en`
2. Confirm the page is usable with `take_snapshot` or `get_page_content`.
   - The usable home state shows the `Ask Scholar` textbox.
3. Try normal submission only if it is clearly working.
   - `fill` on the textbox and `press_key` `Enter` are allowed as a first attempt.
   - If the page does not leave the home state, do not keep retrying blind.
4. Use the JavaScript submission fallback with `evaluate_script`.
   - Set the textarea `#gs_as_i_t` value using the native setter.
   - Dispatch an `input` event.
   - Dispatch a click on `#gs_as_i_s`.
   - This is the path that reliably started a Scholar Labs session in practice.
   - Use this exact shape:

```js
(() => {
  const t = document.getElementById('gs_as_i_t');
  const s = document.getElementById('gs_as_i_s');
  if (!t || !s) return 'missing';
  const set = Object.getOwnPropertyDescriptor(
    HTMLTextAreaElement.prototype,
    'value'
  ).set;
  set.call(t, QUERY);
  t.dispatchEvent(new Event('input', { bubbles: true }));
  s.dispatchEvent(
    new MouseEvent('click', { bubbles: true, cancelable: true, view: window })
  );
  return 'submitted';
})()
```

   - Replace `QUERY` with the actual search string before calling `evaluate_script`.
5. Wait outside BrowserOS before polling.
   - Use `exec_command` with `sleep 6` or `sleep 8`.
   - Then inspect the page again with `take_snapshot` and `get_page_content`.
6. Poll until one of these states is visible:
   - Results cards are visible.
   - `Found 0 relevant results`.
   - A blocked state.
7. For a new search in the same tab:
   - Use `take_snapshot` to get the current element id for `New session`.
   - Use `click` on that element.
   - Confirm the home state is back.
   - Submit the next query.

## Handoff State Machine

Use this exact state flow:

`start -> blocked -> await_user -> resume | abort`

Treat the page as `blocked` when any of the following is true:

- Google sign-in is visible.
- CAPTCHA or unusual-traffic page is visible.

When blocked, send this exact prompt:

`Google Scholar Labs is blocked in BrowserOS. Complete the page in BrowserOS until the results list is visible, then reply 'resume'. Reply 'abort' to stop.`

After the user replies `resume`:

- Inspect the current BrowserOS page.
- Do not reissue the query yet.
- Continue only if visible results cards are present.

After the user replies `abort`:

- Stop and report that Scholar Labs could not be accessed.

If the page is still blocked after one `resume`:

- Abort instead of looping.

If query submission fails but the page is not blocked:

- Treat this as a tooling issue, not as `blocked`.
- Use the `evaluate_script` fallback instead of asking the user to intervene.
- Only stop if BrowserOS cannot open or inspect the page at all.

## Extraction Rules

Read only the first visible results page.

For each visible result card, capture when present:

- Title
- Authors
- Year
- Venue or source
- Scholar link
- Short annotation
- Citation signal
- Position on page

If the page shows fewer than 10 visible cards, use the smaller set.

Use both BrowserOS views during extraction:

- `take_snapshot` for clickable elements, positions, and visible result boundaries.
- `get_page_content` for title, authors, annotation, citation signal, and outbound links.

When `get_page_content` contains loading text such as `Found early results. Searching deeper...` or `Looking for more results...`:

- Use the currently visible cards only.
- Do not wait for full pagination or exhaustive ranking.

If a card is incomplete, keep the missing fields empty and continue.

If the same paper appears more than once as obvious duplicate versions:

- Match by normalized title plus year.
- Keep the higher-positioned card.
- If position is equal or unclear, keep the card with richer metadata.

## Query Construction

If the input is already a keyword query, do not expand it.

If the input is a short task statement, rewrite it once into a compact query:

- Keep the main topic terms.
- Keep one method or phenomenon term when helpful.
- Avoid Boolean logic unless quoting an exact phrase is necessary.

If the first query yields zero usable results:

- Remove the weakest qualifier.
- Retry exactly once.

If the retry also yields zero usable results:

- Return `0 papers found` and stop.

Practical note:

- Scholar Labs handled compact noun-phrase queries better than abstract meta-queries during testing.
- If the first query is too conceptual, drop terms like `longitudinal`, `spatial`, or `temporal` before dropping the main topic.
