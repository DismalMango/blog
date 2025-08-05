---
title: Reducer
date: 2025-07-15
category: 'react'
---

# Reducer

https://zh-hans.react.dev/reference/react/useReducer#dispatch

useReducer

```react
  const [{ cities, isLoading, currentCity }, dispatch] = useReducer(
    reducer,
    initialState
  );
```

```react
const [state, dispatch] = useReducer(reducer, initialArg, init?)
```

```react
function reducer(state, action) {
  switch (action.type) {
    case "loading":
      return {
        ...state,
        isLoading: true,
      };

    case "cities/loaded":
      return {
        ...state,
        isLoading: false,
        cities: action.payload,
      };

    case "city/loaded":
      return {
        ...state,
        isLoading: false,
        currentCity: action.payload,
      };

    case "city/created":
      return {
        ...state,
        isLoading: false,
        cities: [...state.cities, action.payload],
      };

    case "city/deleted":
      return {
        ...state,
        isLoading: false,
        cities: state.cities.filter((city) => city.id !== action.payload),
      };

    case "rejected":
      return {
        ...state,
        isLoading: false,
        error: action.payload,
      };

    default:
      throw new Error("Unknown action type");
  }
}
```

```react
const initialState = {
  cities: [],
  isLoading: false,
  currentCity: {},
  error: "",
};
```

```react
dispatch({ type: "loading" });
```
