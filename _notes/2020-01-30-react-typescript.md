---
layout: post
title: React+Typescript notes
---

# React & Typescrypt

Notes taken from the collective knowledge.
Last update: 14-01-2022

## React properties

- Javascript library (the V in MVC)
- Declarative
- Component-based
  - component manages their own state
  - think application in terms of state rather than explicit manipulation
  - component represent a state via UI
  - one-way data flow
    - from parent to children
    - top-down data flow
  - component === state machine
  - component === function (pure)
    - take parameters and returns a UI output

### Functions

```js
function cloneFilter(filters: FiltersMap): FiltersMap {
  const cloned: FiltersMap = new Map();
  filters.forEach((value, key) => cloned.set(key, value));

  return cloned;
}
```

### Arrow functions

From

```js
function plusOne(x) {
  return x + 1;
}
```

to

```js
const plusOne = (x) => x + 1;
```

### destructuring

```js
let [x, y] = [1, 2];
```

### As simple function component

```js
function Hello() {
	return <div>Hello!</div>
}

export default function Hello() {
  return (
  	<>
		<div>Hello!</div>
		<div>Hello!</div>
	</>
}
```

### Without JSX

```js
const myelement = React.createElement("h1", {}, "I do not use JSX!");
ReactDOM.render(myelement, document.getElementById("root"));
```

### WIth JSX (as syntactic sugar)

```js
const myelement = <h1>I Love JSX!</h1>;
ReactDOM.render(myelement, document.getElementById("root"));
```

### Props are read-only into scope

```js
function Hello(props) {
  return <div>Hello {props.name}!</div>;
}
render(<Hello name="Nicola" />);

function Parent() {
  return <Hello name="Nicola" />;
}
```

### Fragment

List of component without a div container

```js
<>...</>
```

### Inline styling

{% raw %}
```js
function House() {
  const [color, setColor] = useState("red");
  return (
    <div>
      <h2 style={{ border: `1px solid ${color}` }}>This is a {color} house</h2>
    </div>
  );
}
```
{% endraw %}

or

```js
const style = {
  border: `1px solid ${color}`,
};

return (
  <div>
    <h2 style={style}>This is a {color} house</h2>
  </div>
);
```

### Class vs. Functional

- Class Based Component -> mutating state
- Functional Component -> not-mutating state

## Thinking in React

### Workflow

1. draw boxes around every component (and subcomponent) in the mock and give them all names
2. how do you know what should be its own component? Use the single responsibility principle

3. create a static version of the app

   - start with a static version that takes your data model and renders the UI but has no interactivity (decouple these processes because building a static version requires a lot of typing and no thinking, and adding interactivity requires a lot of thinking and not a lot of typing)

   - begin with a version of your app that renders your data model, reuse components and pass data using props (passing data from parent to child)
   - don’t use state at all to build this static version: tate is reserved only for interactivity

   - figure out the absolute minimal representation of the state your application needs and compute everything else you need on-demand.

   - at the end of this step, you’ll have a library of reusable components that render your data model; the components will only have render() methods since this is a static version of your app

4. next, we need to identify which component mutates, or owns, this state
   - React is all about one-way data flow down the component hierarchy
   - identify every component that renders something based on that state
   - find a common owner component (a single component above all the components that need the state in the hierarchy)
   - either the common owner or another component higher up in the hierarchy should own the state
   - if you can’t find a component where it makes sense to own the state, create a new component solely for holding the state and add it somewhere in the hierarchy above the common owner component

### Communication between components

- top to bottom -> props/state
- bottom to top -> callbacks (via props)

## Hooks

Hooks are functions that let you "hook into" React state and lifecycle features from function components.

- only call Hooks at the top level
- only call Hooks from React function components

### `useState()`

- manage local component state

```js
const [counter, setCounter] = useState(0); // destructuring
setCounter(counter + 1); // re-render
```

or

```js
const state = useState(0);
const counter = state[0];
const setCounter = state[1];
```

- if initial state value is heavy to calculate, use 'lazy initial state' approach, passing a function instead of the value

```js
const [data, setData] = useState(() => calculateData(someParameters));
```

- if updated data does not depend on other data, use function to set state allows React to optimize update process (e.g. batching updates, ignoring redundant updates)

```js
const [counter, setCounter] = useState(0);
setCounter((c) => c + 1); // optimized re-render
```

- update complex state

```js
const [data, setData] = useState({name: "Harry", email: "harry.potter@hogwarts.com"});
setData(state => { return {...state, name: "Potter"}}}); // partial update
```

### `useEffect()`

- used to inject behaviours as mount/unmount/changes in functions component
- add side effects to function components (eg. Data fetching, setting up a subscription, and manually changing the DOM in React)
- you tell React that your component needs to do something after render, React will remember the function you passed (we’ll refer to it as our "effect"), and call it later after performing the DOM updates

```js
const [status, setStatus] = useState("");

const loadStatus = useCallback(async () => {
  const [res, data] = await getStatus();
  setStatus(data);
}, []);

useEffect(() => {
  void loadStatus();
}, [loadStatus]);

return <div>{status}</div>;
```

### `memo` (not a hook)

- memorize a component

### `useMemo()`

- memorize a calculated value

```js
const filtersButton = useMemo(() => {
  return <Button onClick={handleClick} />;
}, [handleClick]);
```

### `useCallback()`

- memorize a function definition

```js
const handleApply = useCallback((input: MyType) => {}, [dependencies]);

const onSimProvisionDialogError = useCallback(async () => {
	await ...

  snackbarOpen({
    message: "Cannot load backup list",
    severity: "error",
  });
}, [snackbarOpen]);
```

# Tips

- always emit `null` element in case of empty component (improving performances)

```js
return { has_data && <div>My Data</div> }
```

# Useful info

## Multilanguage support

```js
npm install i18next react-i18next i18next-http-backend i18next-browser-languagedetector --save
```

Create files:

public/assets/i18n/translations/en.json

```json
{
  "Name": "Name"
}
```

public/assets/i18n/translations/fr.json

```json
{
  "Name": "Nom"
}
```

Create file
src/i18n/i18n.js

```js
import i18n from "i18next";
import { initReactI18next } from "react-i18next";

import Backend from "i18next-http-backend";
import LanguageDetector from "i18next-browser-languagedetector";

void i18n
  // https://github.com/i18next/i18next-http-backend
  .use(Backend)
  // detect user language: https://github.com/i18next/i18next-browser-languageDetector
  .use(LanguageDetector)
  .use(initReactI18next)
  // init i18next; other options: https://www.i18next.com/overview/configuration-options
  .init({
    lng: "en",
    backend: { loadPath: "/assets/i18n/{{ns}}/{{lng}}.json" },
    fallbackLng: "en",
    debug: true,
    ns: ["translations"],
    defaultNS: "translations",
    keySeparator: false,
    interpolation: { escapeValue: false, formatSeparator: "," },
    react: { wait: true },
  });

export default i18n;
```

Add to `index.tsx`:

```js
import "./i18n/i18n";
```

create a language selector component `src/components/LanguageSelector/index.tsx`:

```js
import { ChangeEvent, useState } from "react";
import { useTranslation } from "react-i18next";

const LanguageSelector = () => {
  const { i18n } = useTranslation();
  const [selectedLang, setSelectedLang] = useState("en");

  const changeLanguage = (event: ChangeEvent<HTMLInputElement>) => {
    setSelectedLang(event.target.value);
    void i18n.changeLanguage(event.target.value);
  };

  return (
    <div onChange={changeLanguage}>
      <label>
        <input
          type="radio"
          value="en"
          name="language"
          checked={selectedLang === "en"}
        />{" "}
        English
      </label>
      <label>
        <input
          type="radio"
          value="bn"
          name="language"
          checked={selectedLang === "fr"}
        />{" "}
        Français
      </label>
    </div>
  );
};

export default LanguageSelector;
```

Usage:

```js
import { useTranslation } from 'react-i18next';
const { t } = useTranslation();

t('Name')
<div>{t('Name')}</div>
```

or:

```js
import { Trans, withTranslation } from "react-i18next";
<div>
  <Trans count="3">REACT_EXAMPLE</Trans>
</div>;
```

## Suspense

- allow loading (fallback can be null or Component/HTML fragment)

```js
<Suspense fallback={null}>
  <App />
</Suspense>
```

# Debugging

```css
* {
  outline: 1px solid red;
}
```
