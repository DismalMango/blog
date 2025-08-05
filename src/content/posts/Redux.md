---
title: Redux
date: 2025-07-30
category: ''
---

![Screenshot 2025-07-30 at 6.58.25 pm](assets/Screenshot%202025-07-30%20at%206.58.25%E2%80%AFpm.png)

![Screenshot 2025-07-30 at 7.01.28 pm](assets/Screenshot%202025-07-30%20at%207.01.28%E2%80%AFpm.png)

![Screenshot 2025-07-30 at 7.02.46 pm](assets/Screenshot%202025-07-30%20at%207.02.46%E2%80%AFpm.png)

![Screenshot 2025-07-30 at 7.03.37 pm](assets/Screenshot%202025-07-30%20at%207.03.37%E2%80%AFpm.png)

![Screenshot 2025-07-30 at 7.05.02 pm](assets/Screenshot%202025-07-30%20at%207.05.02%E2%80%AFpm.png)

# Action Creator

useSelector

useDispatch

![Screenshot 2025-07-31 at 1.00.17 am](assets/Screenshot%202025-07-31%20at%201.00.17%E2%80%AFam.png)

![Screenshot 2025-07-31 at 1.01.08 am](assets/Screenshot%202025-07-31%20at%201.01.08%E2%80%AFam.png)

## Redux vs. Context API

In this lecture, we compare Redux with React's Context API to understand their differences, advantages, and appropriate use cases.

Many developers have started viewing the Context API, especially when combined with `useReducer`, as a complete replacement for Redux. While this can be true in some situations, it is important not to follow trends blindly but to have a nuanced and fact-based discussion.

This video summarizes what we have learned about both solutions for managing global state by examining their pros and cons, and ultimately provides recommendations on when to use Context and when to use Redux.

### Advantages of Context API + useReducer

- These features are already built into React, so no additional packages are required.
- Using Redux requires installing extra packages, which adds more work and increases bundle size.
- Setting up a single context with `useReducer` is straightforward: pass the state value into the context and provide it to the app.

However, for each additional state slice, you must repeat the setup from scratch. This can lead to "provider hell," where many context providers accumulate in the main app component.

### Advantages of Redux

- Redux requires more initial setup, but adding additional state slices is straightforward.
- To add a new slice, create a new slice file and add reducers; no need to create another provider or custom hook.
- Redux supports middleware, which allows handling asynchronous tasks within the state management tool.

The Context API has no built-in mechanism for asynchronous operations such as data fetching, as experienced in the worldwide application example. Middleware in Redux, although not necessarily easy to use, provides a way to handle asynchronous tasks.

It is important to note that neither tool should be used for managing remote state extensively, but sometimes fetching a single data point from an API can be useful, as demonstrated with currency conversion in this section.

### Performance Considerations

Optimizing the performance of the Context API and `useReducer` solution can require additional work, whereas Redux comes with many optimizations out of the box, including minimizing wasted renders.

### Developer Tools

Redux offers excellent DevTools, while the Context API relies on the simpler React developer tools. This difference can be significant in applications with large, complex, and frequently updated state.

Although Redux appears to have more advantages than the Context API, the goal here is not to count pros and cons but to present facts for informed decision-making.

## Usage Recommendations

The general consensus is to use the Context API plus React hooks for global state management in small applications, and Redux for large applications. However, this advice is not always helpful, so let's explore further.

There is never a single right answer that fits every project. The choice between these solutions depends entirely on the project's needs.

For example, if you only need to share a value that does not change often, the Context API is perfect. Examples include the app color theme, user's preferred language, or the currently authenticated user. These values rarely change, so optimization is not necessary.

On the other hand, if you have lots of UI state that requires frequent updates, such as shopping carts, currently open tabs, or complex data filters, Redux might be the better choice. Redux is heavily optimized for frequent updates.

Redux is also ideal for complex state with nested objects because Redux Toolkit allows mutating state in a helpful way.

These situations where Redux is preferred are not very common for UI state, which explains why Redux has fallen somewhat out of favor recently.

The Context API can be very helpful for solving simple prop drilling problems or managing state in a local sub-tree of the application. This is not truly global state but state global to a smaller sub-part of the app.

An important use case of this is the advanced compound component pattern, which will be studied later.

Ultimately, you need to choose what is best for your particular project and not simply follow trends you read about online. This principle applies to every technological choice you make.

The debate between Context API and Redux has been ongoing in the React community for some time, which is why this discussion is important.

With this, we have finished another section. I hope you now feel confident with Redux, and I look forward to seeing you back here very soon.

## Key Takeaways

- The Context API combined with useReducer is built into React and suitable for simple global state management with infrequent updates.
- Redux requires additional setup but excels in managing complex, frequently updated state with built-in optimizations and middleware support.
- Context API can lead to "provider hell" when managing multiple state slices, whereas Redux simplifies adding new slices without extra providers.
- Choosing between Context API and Redux depends on project needs; avoid following trends blindly and select the tool best suited for your application's complexity and state management requirements.
