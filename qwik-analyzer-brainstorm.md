# Qwik Analyzer

## Problem #1 How to do we find out about the nature of referenced components?

Take this example, `slot_example.tsx`:

```tsx
import { component$ } from "@builder.io/qwik";
import type { DocumentHead } from "@builder.io/qwik-city";
import { MyTest } from "../components/my-test";

export default component$(() => {
	return (
		<>
			<MyTest.Root>
				<MyTest.Child />
			</MyTest.Root>
		</>
	);
});

export const head: DocumentHead = {
	title: "Welcome to Qwik",
	meta: [
		{
			name: "description",
			content: "Qwik site description",
		},
	],
};
```
We need to know more about the `MyTest` component, specifically: the nature of the `Root` and `Child` components.

From OXC's point of view, we know the following:
- `MyTest` is a component that is imported from `../components/my-test`.
- `MyTest` has a unique SymbolID given to it via OXC's semantic analysis. 
- `<Mytest.Root>` and `<MyTest.Child>` are member expressions, both of which have unique Reference IDs that all resolve to the same SymbolID of `MyTest`.

The missing part of the puzzle is how to find out the nature of `Root` and `Child`.  For that we need further analysis of the `MyTest` component.

This can only be achieved by parsing and running semantic analysis on `../components/my-test/index.ts`, which is where the actual definition of `MyTest` resides.

Contents of `../components/my-test/index.ts`

```ts
import { MyTestChild } from "./my-test-child";
import { MyTestRoot } from "./my-test-root";

export const MyTest = {
    Root: MyTestRoot,
    Child: MyTestChild,
};
```
By additionally parsing and analyzing `../components/my-test/index.ts`, we can build a relationship the `MyTest` in `slot_example.tsx` to the `MyTestRoot` and `MyTestChild` components.

Or, at least that is the gist of what we want to accomplish.