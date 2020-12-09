Can you use object spread and destructuring at the same time? For example:

```javascript
const {values} = state,
      newValues = [...values];
```

Can we somehow combine the destructuring and object spread, so that we will only have one line of code which declares a `values` variable which is a copy of the `state.values`? Or is the only way to do it the following:

```javascript
const values = [...state.values];
```
