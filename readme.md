# Browser Navigation Challenge — Finish Route Discovery via Submit Handler Inspection

Challenge solved in < 0.1s. It also took GPT less than 5s to write up the explanation too.

Here:

## Finish it too

1. Navigate to `https://serene-frangipane-7fd25b.netlify.app/`,
2. Open dev tools
3. Run
 ```js
 (() => {
  const params = new URLSearchParams(location.search);
  const version = params.get("version") || "1";
  const target = `/step30?version=${version}`;

  if (window.__reactRouterDataRouter && typeof window.__reactRouterDataRouter.navigate === "function") {
    window.__reactRouterDataRouter.navigate(target, { replace: true });
    return;
  }

  history.pushState(history.state, "", target);
  window.dispatchEvent(new PopStateEvent("popstate"));
})();
```

## Overview

This writeup documents how the hidden `/finish` route was discovered by inspecting the client-side submit handler responsible for validating the final challenge code.

The key breakthrough was identifying that the application stored its completion navigation logic entirely inside a front-end event handler. 

By extracting and analyzing this handler, the finish endpoint was revealed without needing to solve the challenge.

---

## Context

After bypassing step navigation using browser history manipulation, access was gained to the final step of the challenge. The final step required entering a 6-character code before progressing.

At this point, the focus shifted from solving the challenge logic to analyzing how completion was handled internally.

---

## Locating The Submit Handler

Inspection of the page revealed that the final input form used a React-controlled submit event rather than a traditional form submission. Using React Fiber inspection, the form component props were examined.

This revealed the submit handler function:

```
onSubmit: oe(se)
```

This indicated that the function `oe` contained the final validation and navigation logic.

---

## Extracting The Handler Logic

Dumping the function source revealed the following logic:

```js
se => {
  se && se.preventDefault();

  const ne = E.trim().toUpperCase().replace(/\s/g,"");

  Cv(c, ne)
    ? (
        S(!0),
        setTimeout(() => {
          c < Yr
            ? o(`/step${c+1}?version=${u}`)
            : o("/finish")
        }, 500)
      )
    : (
        h(!0),
        b(""),
        setTimeout(() => h(!1), 2000)
      )
}
```

---

## Critical Discovery

The navigation logic clearly revealed the completion route:

```js
o("/finish")
```

This confirmed:

- The application redirected users to `/finish` after completing the final step
- The finish route was not hidden or protected
- Navigation was triggered purely by client-side logic

---

## Understanding The Handler

Breaking down the function:

### Input Normalization

```js
const ne = E.trim().toUpperCase().replace(/\s/g,"");
```

The entered code was normalized by:

- Removing whitespace
- Converting to uppercase
- Trimming input

---

### Code Validation

```js
Cv(c, ne)
```

This function verified whether the code matched the expected value for the current step.

---

### Completion Navigation

```js
c < Yr
  ? o(`/step${c+1}?version=${u}`)
  : o("/finish")
```

This logic revealed that:

- If the current step was not the final step, navigation continued to the next step
- If the current step equaled the final step, navigation redirected to `/finish`

This line directly exposed the hidden finish endpoint.

---

## Verifying Direct Access

After discovering the route, it was confirmed that `/finish` could be accessed directly using browser history manipulation.

```js
history.pushState(history.state, "", "/finish");
window.dispatchEvent(new PopStateEvent("popstate"));
```

This successfully loaded the completion page without solving any challenge logic.

---

## Root Vulnerability

The application exposed critical routing information inside client-side event handlers. Because navigation targets were embedded directly in JavaScript logic, inspecting runtime handlers revealed hidden endpoints.

There was:

- No server-side verification
- No access control protecting the finish route
- Full reliance on UI-triggered navigation

---

## Security Lessons

### Never Store Sensitive Routes Only In Client Logic

Any navigation endpoint stored in front-end code can be extracted through runtime inspection.

---

### Client Validation Must Not Control Access

Completion logic must be validated server-side before allowing progression.

---

### Obfuscation Does Not Provide Security

Minified function names such as `oe` and `Cv` do not prevent analysis or reverse engineering.

---

## Conclusion

The `/finish` route was discovered by inspecting the React submit handler `oe`, which contained explicit navigation logic for final challenge completion. Once identified, the route was directly accessible using browser history manipulation, bypassing all challenge r
