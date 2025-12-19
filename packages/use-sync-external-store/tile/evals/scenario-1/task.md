# Shopping Cart State Manager

## Goal

Build a shopping cart state manager that subscribes to an external store and efficiently renders cart information using advanced memoization and selector patterns.

## Background

You are building a component for an e-commerce site that displays shopping cart information. The cart data is stored in an external store that can be updated from multiple sources (user actions, API responses, etc.). You need to ensure your component subscribes to this store efficiently and only re-renders when relevant data changes.

## Requirements

### 1. External Store Implementation

Create a store that manages shopping cart state with the following structure:

```javascript
{
  items: [
    { id: string, name: string, price: number, quantity: number },
    ...
  ],
  lastUpdated: number  // timestamp
}
```

The store must:
- Maintain state in a mutable object
- Support subscribing to changes with a callback function
- Return an unsubscribe function from the subscribe method
- Provide a method to get the current snapshot
- Notify all subscribers when state changes
- Support methods to add items, remove items, and update quantities

### 2. Cart Summary Component

Create a `CartSummary` component that displays:
- Total number of items in the cart (sum of all quantities)
- Total price (sum of price × quantity for all items)

This component must:
- Use the selector pattern to extract only the needed data
- Implement a custom equality function to prevent unnecessary re-renders
- Only re-render when the total count or total price actually changes (not when lastUpdated changes)

### 3. Individual Item Component

Create a `CartItem` component that displays a single item's details (name, price, quantity).

This component must:
- Accept an item ID as a prop
- Subscribe to the store and select only the specific item's data
- Use a custom equality function to prevent re-renders when other items change
- Handle the case when the item is removed from the cart

## Test Cases { .tests }

### Test Case 1: Basic Store Operations { .test }

**File**: `store.test.js`

```javascript
test('store manages cart state and notifies subscribers', () => {
  const store = createCartStore();
  const callback = jest.fn();

  const unsubscribe = store.subscribe(callback);

  store.addItem({ id: '1', name: 'Widget', price: 10, quantity: 1 });
  expect(callback).toHaveBeenCalledTimes(1);

  store.updateQuantity('1', 2);
  expect(callback).toHaveBeenCalledTimes(2);

  unsubscribe();
  store.removeItem('1');
  expect(callback).toHaveBeenCalledTimes(2); // Not called after unsubscribe
});
```

### Test Case 2: CartSummary Re-render Optimization { .test }

**File**: `CartSummary.test.js`

```javascript
test('CartSummary only re-renders when totals change', () => {
  const store = createCartStore();
  let renderCount = 0;

  function CartSummaryWithSpy() {
    const summary = useCartSummary(store);
    renderCount++;
    return <div>{summary.itemCount} items, ${summary.totalPrice}</div>;
  }

  render(<CartSummaryWithSpy />);
  expect(renderCount).toBe(1);

  // Add item - should re-render
  act(() => store.addItem({ id: '1', name: 'Widget', price: 10, quantity: 1 }));
  expect(renderCount).toBe(2);

  // Update lastUpdated only - should NOT re-render
  act(() => store.updateTimestamp());
  expect(renderCount).toBe(2);

  // Update quantity - should re-render
  act(() => store.updateQuantity('1', 2));
  expect(renderCount).toBe(3);
});
```

### Test Case 3: CartItem Isolation { .test }

**File**: `CartItem.test.js`

```javascript
test('CartItem only re-renders when its specific item changes', () => {
  const store = createCartStore();
  store.addItem({ id: '1', name: 'Widget A', price: 10, quantity: 1 });
  store.addItem({ id: '2', name: 'Widget B', price: 20, quantity: 1 });

  let renderCount = 0;

  function CartItemWithSpy({ itemId }) {
    const item = useCartItem(store, itemId);
    renderCount++;
    if (!item) return <div>Item not found</div>;
    return <div>{item.name} - ${item.price} x {item.quantity}</div>;
  }

  render(<CartItemWithSpy itemId="1" />);
  expect(renderCount).toBe(1);

  // Update item 2 - should NOT re-render item 1
  act(() => store.updateQuantity('2', 3));
  expect(renderCount).toBe(1);

  // Update item 1 - should re-render
  act(() => store.updateQuantity('1', 2));
  expect(renderCount).toBe(2);
});
```

## Constraints

- Do not use external state management libraries (Redux, Zustand, etc.)
- The store must be a plain JavaScript object/class with mutable state
- Components must subscribe to the external store, not use React state as the source of truth
- Selector functions may be defined inline (unmemoized) in components
- Custom equality functions must perform structural comparison, not reference equality

## Deliverables

1. `cartStore.js` - The external store implementation
2. `CartSummary.js` - Component using selector pattern for cart totals
3. `CartItem.js` - Component subscribing to individual item data
4. Test files demonstrating the re-render optimization behavior

## Dependencies { .dependencies }

### use-sync-external-store { .dependency }

Provides hooks for subscribing to external stores with React compatibility.
