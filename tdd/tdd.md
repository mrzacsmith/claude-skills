# Test-Driven Development

Build features or fix bugs using red-green-refactor TDD with vertical slices.

## Philosophy

**Core principle**: Tests verify behavior through public interfaces, not implementation details. Code can change entirely; tests shouldn't.

**Good tests** are integration-style: they exercise real code paths through public APIs. They describe _what_ the system does, not _how_. A good test reads like a specification — "user can checkout with valid cart" tells you exactly what capability exists. These tests survive refactors because they don't care about internal structure.

**Bad tests** are coupled to implementation. They mock internal collaborators, test private methods, or verify through external means. Warning sign: your test breaks when you refactor, but behavior hasn't changed.

## Anti-Pattern: Horizontal Slices

**DO NOT write all tests first, then all implementation.** This produces crap tests — tests written in bulk test _imagined_ behavior, not _actual_ behavior.

```
WRONG (horizontal):
  RED:   test1, test2, test3, test4, test5
  GREEN: impl1, impl2, impl3, impl4, impl5

RIGHT (vertical):
  RED->GREEN: test1->impl1
  RED->GREEN: test2->impl2
  RED->GREEN: test3->impl3
```

---

## Workflow

### 1. Planning

- Confirm with user what interface changes are needed
- Confirm which behaviors to test (prioritize)
- Identify opportunities for deep modules (small interface, deep implementation)
- Design interfaces for testability
- List the behaviors to test (not implementation steps)
- Get user approval on the plan

**Wait for user approval before writing any code.**

### 2. Tracer Bullet

Write ONE test that confirms ONE thing about the system:

```
RED:   Write test for first behavior -> test fails
GREEN: Write minimal code to pass -> test passes
```

### 3. Incremental Loop

For each remaining behavior:

```
RED:   Write next test -> fails
GREEN: Minimal code to pass -> passes
```

Rules:
- One test at a time
- Only enough code to pass current test
- Don't anticipate future tests
- Keep tests focused on observable behavior

### 4. Refactor

After all tests pass, look for refactor candidates:
- Extract duplication
- Deepen modules
- Apply SOLID principles
- Consider what new code reveals about existing code
- Run tests after each refactor step

**Never refactor while RED.**

---

## When to Mock

Mock at **system boundaries** only:

- External APIs (payment, email, Firebase services in some cases)
- Time/randomness
- File system (sometimes)

**Don't mock:**

- Your own classes/modules
- Internal collaborators
- Anything you control

Use dependency injection to make system boundaries mockable:

```javascript
// Easy to mock
function processPayment(order, paymentClient) {
  return paymentClient.charge(order.total);
}

// Hard to mock
function processPayment(order) {
  const client = new StripeClient(process.env.STRIPE_KEY);
  return client.charge(order.total);
}
```

## Good vs Bad Tests

```javascript
// GOOD: Tests observable behavior through public interface
test("user can checkout with valid cart", async () => {
  const cart = createCart();
  cart.add(product);
  const result = await checkout(cart, paymentMethod);
  expect(result.status).toBe("confirmed");
});

// BAD: Tests implementation details
test("checkout calls paymentService.process", async () => {
  const mockPayment = jest.mock(paymentService);
  await checkout(cart, payment);
  expect(mockPayment.process).toHaveBeenCalledWith(cart.total);
});

// BAD: Bypasses interface to verify
test("createUser saves to database", async () => {
  await createUser({ name: "Alice" });
  const row = await db.query("SELECT * FROM users WHERE name = ?", ["Alice"]);
  expect(row).toBeDefined();
});

// GOOD: Verifies through interface
test("createUser makes user retrievable", async () => {
  const user = await createUser({ name: "Alice" });
  const retrieved = await getUser(user.id);
  expect(retrieved.name).toBe("Alice");
});
```

---

## Checklist Per Cycle

- [ ] Test describes behavior, not implementation
- [ ] Test uses public interface only
- [ ] Test would survive internal refactor
- [ ] Code is minimal for this test
- [ ] No speculative features added

---

## Rules

1. **Always get user approval** on the plan before writing code
2. **One test at a time** — vertical slices, never horizontal
3. **Never refactor while RED** — all tests must pass first
4. **Tests verify behavior** not implementation
5. **Mock only at system boundaries**
