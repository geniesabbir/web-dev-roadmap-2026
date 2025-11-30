# TypeScript

**Duration:** 2 weeks

## Learning Objectives

By the end of this section, you will:
- Understand why TypeScript exists and its benefits
- Write type-safe JavaScript code
- Use interfaces and type aliases effectively
- Work with generics
- Configure TypeScript projects
- Integrate TypeScript with your workflow

---

## Week 1: TypeScript Fundamentals

### Day 1: Getting Started
**Topics:**
- What is TypeScript and why use it?
- Installing TypeScript
- `tsc` compiler basics
- `tsconfig.json` configuration
- Type inference

**Setup:**
```bash
# Install TypeScript globally
npm install -g typescript

# Initialize a TypeScript project
tsc --init
```

**Exercises:**
```typescript
// exercises/01-basics.ts

// 1. Set up a TypeScript project
// 2. Compile your first .ts file
// 3. Experiment with type inference
```

### Day 2: Basic Types
**Topics:**
- Primitive types (`string`, `number`, `boolean`)
- Arrays and tuples
- `any`, `unknown`, `never`, `void`
- Type annotations
- Union types (`|`)
- Literal types

**Exercises:**
```typescript
// exercises/02-basic-types.ts

// 1. Type a user profile object
// 2. Create a function with typed parameters
// 3. Use union types for flexible inputs
// 4. Practice with tuples
```

### Day 3: Interfaces
**Topics:**
- Defining interfaces
- Optional properties (`?`)
- Readonly properties
- Extending interfaces
- Interface vs Type alias
- Index signatures

**Exercises:**
```typescript
// exercises/03-interfaces.ts

// 1. Define an interface for a Product
// 2. Extend it for different product types
// 3. Create optional and readonly properties
// 4. Use index signatures for dynamic keys
```

### Day 4: Type Aliases & Advanced Types
**Topics:**
- Type aliases
- Intersection types (`&`)
- Type narrowing
- Type guards (`typeof`, `instanceof`, `in`)
- Discriminated unions
- Type assertions (`as`)

**Exercises:**
```typescript
// exercises/04-advanced-types.ts

// 1. Create discriminated unions for shapes
// 2. Write type guard functions
// 3. Practice type narrowing
// 4. Use intersection types to combine types
```

### Day 5: Functions
**Topics:**
- Function type annotations
- Optional and default parameters
- Rest parameters with types
- Function overloads
- `this` parameter
- Callback types

**Exercises:**
```typescript
// exercises/05-functions.ts

// 1. Type a function that fetches data
// 2. Create function overloads
// 3. Type callback functions
// 4. Build a typed event handler
```

---

## Week 2: Advanced TypeScript

### Day 1: Generics
**Topics:**
- Why generics?
- Generic functions
- Generic interfaces
- Generic classes
- Generic constraints (`extends`)
- Default type parameters

**Exercises:**
```typescript
// exercises/06-generics.ts

// 1. Create a generic identity function
// 2. Build a generic Stack class
// 3. Type a generic API response wrapper
// 4. Use constraints for specific requirements
```

### Day 2: Utility Types
**Topics:**
- `Partial<T>`
- `Required<T>`
- `Pick<T, K>`
- `Omit<T, K>`
- `Record<K, T>`
- `Readonly<T>`
- `ReturnType<T>`
- `Parameters<T>`

**Exercises:**
```typescript
// exercises/07-utility-types.ts

// 1. Use Partial for update functions
// 2. Pick specific properties for DTOs
// 3. Create a typed dictionary with Record
// 4. Extract function types with ReturnType
```

### Day 3: Classes & OOP
**Topics:**
- Class with types
- Access modifiers (`public`, `private`, `protected`)
- Abstract classes
- Implementing interfaces
- Static properties and methods
- Parameter properties

**Exercises:**
```typescript
// exercises/08-classes.ts

// 1. Create a typed BankAccount class
// 2. Use access modifiers appropriately
// 3. Implement an interface with a class
// 4. Build an abstract base class
```

### Day 4: Modules & Declaration Files
**Topics:**
- ES modules in TypeScript
- Type-only imports/exports
- Declaration files (`.d.ts`)
- `@types` packages
- Ambient declarations
- Module augmentation

**Exercises:**
```typescript
// exercises/09-modules/

// 1. Create a typed module system
// 2. Write a declaration file for a JS library
// 3. Use @types packages
```

### Day 5: TypeScript with DOM
**Topics:**
- DOM types (`HTMLElement`, `HTMLInputElement`, etc.)
- Event types (`MouseEvent`, `KeyboardEvent`, etc.)
- Type assertions with DOM
- Null checking with DOM elements
- Generic DOM methods

**Exercises:**
```typescript
// exercises/10-dom/index.ts

// 1. Type a form handler
// 2. Create typed event listeners
// 3. Build a typed DOM utility library
```

---

## Projects

### Mini Project: TypeScript Todo App
Refactor a JavaScript todo app to TypeScript:
- Type all functions and variables
- Create interfaces for Todo items
- Use generics for storage utility
- Handle null checks properly
- Use strict mode

### Configuration Deep Dive
Understand these `tsconfig.json` options:
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

---

## Common Patterns

### API Response Handling
```typescript
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

interface User {
  id: number;
  name: string;
  email: string;
}

async function fetchUser(id: number): Promise<ApiResponse<User>> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}
```

### Event Handler Types
```typescript
type ClickHandler = (event: MouseEvent) => void;
type InputHandler = (event: Event & { target: HTMLInputElement }) => void;

const handleClick: ClickHandler = (e) => {
  console.log(e.clientX, e.clientY);
};
```

### State Management Pattern
```typescript
type State = {
  user: User | null;
  loading: boolean;
  error: string | null;
};

type Action =
  | { type: 'FETCH_START' }
  | { type: 'FETCH_SUCCESS'; payload: User }
  | { type: 'FETCH_ERROR'; payload: string };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'FETCH_START':
      return { ...state, loading: true, error: null };
    case 'FETCH_SUCCESS':
      return { ...state, loading: false, user: action.payload };
    case 'FETCH_ERROR':
      return { ...state, loading: false, error: action.payload };
  }
}
```

---

## Resources

### Documentation
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/)
- [TypeScript Playground](https://www.typescriptlang.org/play)

### Practice
- [Type Challenges](https://github.com/type-challenges/type-challenges)
- [Exercism TypeScript Track](https://exercism.org/tracks/typescript)

### Tools
- [ts-node](https://github.com/TypeStrong/ts-node) - Run TS directly
- [tsx](https://github.com/esbuild-kit/tsx) - Faster ts-node alternative

---

## Checklist Before Moving On

- [ ] Can set up a TypeScript project from scratch
- [ ] Understand basic types and type inference
- [ ] Can create and use interfaces
- [ ] Understand union and intersection types
- [ ] Can use type guards for narrowing
- [ ] Understand and can use generics
- [ ] Know the common utility types
- [ ] Can type DOM operations
- [ ] Can type functions including callbacks
- [ ] Completed the TypeScript Todo project

---

**Next:** [Git & GitHub](../04-git/README.md)
