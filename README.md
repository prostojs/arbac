# @prostojs/arbac

Advanced Role-Based Access Control (ARBAC) for Node.js, designed to facilitate the implementation of complex access control systems in a straightforward and intuitive manner. With `@prostojs/arbac`, you can define roles, resources, and rules with dynamic scopes based on user attributes, enabling fine-grained access control tailored to your application's specific needs.

## Features

- Define roles with customizable rules.
- Control access based on user attributes.
- Support for dynamic scopes to fine-tune access control.
- Wildcard support in resource names and actions for flexible rule definition.
- Easy integration with any Node.js application.

## Installation

Install `@prostojs/arbac` using npm:

```sh
npm install @prostojs/arbac
```

Or using yarn:

```sh
yarn add @prostojs/arbac
```

## Quick Start

Below is a quick example to demonstrate the basic setup and usage:

```typescript
import { Arbac } from '@prostojs/arbac';

// Define your user attributes and scope types
interface UserAttributes {
  userId: string;
  departmentId: string[];
}

interface AccessScope {
  departmentIds: string[];
}

// Initialize ARBAC
const arbac = new Arbac<UserAttributes, AccessScope>();

arbac.registerRole({
  id: 'employee',
  rules: [
    {
      action: 'read',
      resource: 'com.resource.db.leads',
      scope: userAttrs => ({ departmentIds: userAttrs.departmentId }),
    },
  ],
});

// Define a user
const user = {
  id: 'user123',
  roles: ['employee'],
  attrs: async (userId) => ({
    userId,
    departmentId: ['dept1', 'dept2'],
  }),
};

// Evaluate access
const result = await arbac.evaluate({
  resource: 'com.resource.db.leads',
  action: 'read',
}, user);

console.log(result); // { allowed: true, scopes: [{ departmentIds: ['dept1', 'dept2'] }] }
```

## Scopes

Scopes allow defining fine-grained access control rules by dynamically assigning context-based restrictions. When a role grants access to a resource, it can specify a `scope` function that derives additional constraints based on the user's attributes. These scopes are evaluated dynamically and returned as part of the evaluation result.

For example, a role may grant access to a resource but restrict access to specific departments the user belongs to:

```ts
arbac.registerRole({
  id: 'com.role.sales',
  rules: [
    {
      action: 'read',
      resource: 'com.resource.db.leads',
      scope: userAttrs => ({ departmentIds: userAttrs.departmentId }),
    },
  ],
});
```

If a user with `com.role.sales` has `{ departmentId: ['dept1', 'dept2'] }`, the evaluation will return:

```json
{
  "allowed": true,
  "scopes": [{ "departmentIds": ["dept1", "dept2"] }]
}
```

Scopes are useful when access should be limited based on user-specific attributes such as organizational structure, project assignments, or other contextual factors.

## Role and Resource Management

### Registering Roles

```ts
arbac.registerRole({
  id: 'com.role.editor',
  rules: [
    {
      action: 'edit',
      resource: 'com.resource.docs.*',
    },
    {
      action: 'delete',
      effect: 'deny',
      resource: 'com.resource.docs.archived',
    },
  ],
});
```

### Resource Registration

Resources are automatically registered when evaluated. However, you can manually register them if needed:

```ts
arbac.registerResource('com.resource.docs.article');
```

## Evaluation

```ts
const result = await arbac.evaluate(
  {
    action: 'edit',
    resource: 'com.resource.docs.article',
  },
  {
    id: 'user123',
    roles: ['com.role.editor'],
    attrs: { userId: 'user123', departmentId: ['dept1', 'dept2'] },
  }
);

console.log(result); // { allowed: true }
```

## Wildcards in Rules

- `*` matches any single segment (e.g., `com.resource.db.*` allows access to `com.resource.db.user` but not `com.resource.api.user`).
- `**` matches multiple segments (e.g., `com.resource.**` allows access to `com.resource.db.user` and `com.resource.api.user`).

### Example with Wildcards

```typescript
arbac.registerRole({
  id: 'admin',
  rules: [
    {
      action: '*',
      resource: 'com.resource.db.*',
    },
    {
      action: 'delete',
      effect: 'deny',
      resource: 'com.resource.db.sensitiveData',
    },
    {
      action: '*',
      resource: 'com.resource.**',
    },
  ],
});
```

## API Reference

### `Arbac<TUserAttrs, TScope>`

Main class to manage roles, resources, and access evaluation.

#### `constructor()`

Initializes the ARBAC system.

#### `registerRole(role: TArbacRole<TUserAttrs, TScope>): void`

Registers a new role with the ARBAC system.

#### `registerResource(resource: string): void`

Registers a new resource. This is optional as resources are auto-registered when evaluated.

#### `async evaluate(opts: { resource: string; action: string }, user: { id: string; roles: string[]; attrs: TUserAttrs | ((userId: string) => Promise<TUserAttrs>) }): Promise<TArbacEvalResult<TScope>>`

Evaluates access for a given user, resource, and action. Returns an object indicating whether access is allowed and any applicable scopes.

## Testing

Run tests using `vitest`:

```sh
npm test
```

## Contributing

Contributions are welcome! Please feel free to submit a pull request or open an issue to discuss proposed changes or enhancements.

## License

This project is licensed under the [MIT License](LICENSE).

