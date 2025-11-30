# Day 6: Lists & Keys

## Rendering Lists in React

Lists are fundamental to most applications - displaying products, users, messages, etc. React uses JavaScript's `map()` function to transform arrays into lists of elements.

### Basic List Rendering

```jsx
function SimpleList() {
    const fruits = ['Apple', 'Banana', 'Cherry', 'Date'];

    return (
        <ul>
            {fruits.map((fruit, index) => (
                <li key={index}>{fruit}</li>
            ))}
        </ul>
    );
}
```

### Rendering Objects

```jsx
function UserList() {
    const users = [
        { id: 1, name: 'Alice', email: 'alice@example.com' },
        { id: 2, name: 'Bob', email: 'bob@example.com' },
        { id: 3, name: 'Charlie', email: 'charlie@example.com' }
    ];

    return (
        <div>
            {users.map(user => (
                <div key={user.id} className="user-card">
                    <h3>{user.name}</h3>
                    <p>{user.email}</p>
                </div>
            ))}
        </div>
    );
}
```

---

## Understanding Keys

Keys help React identify which items have changed, been added, or removed. They give elements a stable identity.

### Why Keys Matter

```jsx
// Without proper keys, React can't efficiently update lists
// When you add/remove items, React may:
// - Re-render all items instead of just changed ones
// - Lose component state
// - Cause incorrect animations

// Example of problem:
function BadExample() {
    const [items, setItems] = useState(['A', 'B', 'C']);

    const addFirst = () => {
        setItems(['New', ...items]);
    };

    // Using index as key - BAD!
    return (
        <ul>
            {items.map((item, index) => (
                <li key={index}>
                    <input defaultValue={item} />
                </li>
            ))}
        </ul>
    );
    // Adding item at beginning causes all inputs to show wrong values!
}
```

### Key Rules

```jsx
// ✅ GOOD: Use unique, stable IDs
{items.map(item => (
    <div key={item.id}>{item.name}</div>
))}

// ✅ GOOD: UUID or database IDs
{items.map(item => (
    <div key={item.uuid}>{item.name}</div>
))}

// ⚠️ OK: Index as key ONLY when:
// - List is static (never changes)
// - Items have no stable IDs
// - List will not be reordered/filtered
{staticItems.map((item, index) => (
    <div key={index}>{item}</div>
))}

// ❌ BAD: Random keys (regenerated each render)
{items.map(item => (
    <div key={Math.random()}>{item.name}</div>  // Never do this!
))}

// ❌ BAD: Index for dynamic lists with state
{dynamicItems.map((item, index) => (
    <InputComponent key={index} />  // Will lose state when list changes
))}
```

### Keys Must Be Unique Among Siblings

```jsx
function TwoLists() {
    const fruits = [{ id: 1, name: 'Apple' }, { id: 2, name: 'Banana' }];
    const vegetables = [{ id: 1, name: 'Carrot' }, { id: 2, name: 'Broccoli' }];

    return (
        <div>
            {/* These can have same keys - different arrays */}
            <ul>
                {fruits.map(fruit => (
                    <li key={fruit.id}>{fruit.name}</li>
                ))}
            </ul>

            <ul>
                {vegetables.map(veg => (
                    <li key={veg.id}>{veg.name}</li>
                ))}
            </ul>
        </div>
    );
}
```

---

## Extracting List Components

### Component Per Item

```jsx
// Item component
function ProductCard({ product, onAddToCart }) {
    return (
        <div className="product-card">
            <img src={product.image} alt={product.name} />
            <h3>{product.name}</h3>
            <p>${product.price}</p>
            <button onClick={() => onAddToCart(product.id)}>
                Add to Cart
            </button>
        </div>
    );
}

// List component
function ProductList({ products, onAddToCart }) {
    if (products.length === 0) {
        return <p>No products found</p>;
    }

    return (
        <div className="product-grid">
            {products.map(product => (
                <ProductCard
                    key={product.id}
                    product={product}
                    onAddToCart={onAddToCart}
                />
            ))}
        </div>
    );
}
```

### Where to Put Keys

```jsx
// ❌ Wrong: Key on the wrong element
function ListItem({ item }) {
    return <li key={item.id}>{item.text}</li>;  // Key here is ignored
}

function List({ items }) {
    return (
        <ul>
            {items.map(item => (
                <ListItem item={item} />  // Key should be here!
            ))}
        </ul>
    );
}

// ✅ Correct: Key on the component in the array
function ListItem({ item }) {
    return <li>{item.text}</li>;  // No key needed here
}

function List({ items }) {
    return (
        <ul>
            {items.map(item => (
                <ListItem key={item.id} item={item} />  // Key on the array element
            ))}
        </ul>
    );
}
```

---

## List Operations

### Filtering Lists

```jsx
function FilteredList() {
    const [filter, setFilter] = useState('all');
    const [tasks, setTasks] = useState([
        { id: 1, text: 'Learn React', completed: true },
        { id: 2, text: 'Build project', completed: false },
        { id: 3, text: 'Deploy app', completed: false }
    ]);

    const filteredTasks = tasks.filter(task => {
        if (filter === 'all') return true;
        if (filter === 'completed') return task.completed;
        if (filter === 'active') return !task.completed;
        return true;
    });

    return (
        <div>
            <div>
                <button onClick={() => setFilter('all')}>All</button>
                <button onClick={() => setFilter('active')}>Active</button>
                <button onClick={() => setFilter('completed')}>Completed</button>
            </div>

            <ul>
                {filteredTasks.map(task => (
                    <li key={task.id}>{task.text}</li>
                ))}
            </ul>
        </div>
    );
}
```

### Sorting Lists

```jsx
function SortableList() {
    const [sortBy, setSortBy] = useState('name');
    const [sortOrder, setSortOrder] = useState('asc');

    const users = [
        { id: 1, name: 'Charlie', age: 25 },
        { id: 2, name: 'Alice', age: 30 },
        { id: 3, name: 'Bob', age: 22 }
    ];

    const sortedUsers = [...users].sort((a, b) => {
        let comparison = 0;

        if (sortBy === 'name') {
            comparison = a.name.localeCompare(b.name);
        } else if (sortBy === 'age') {
            comparison = a.age - b.age;
        }

        return sortOrder === 'asc' ? comparison : -comparison;
    });

    const toggleSort = (field) => {
        if (sortBy === field) {
            setSortOrder(prev => prev === 'asc' ? 'desc' : 'asc');
        } else {
            setSortBy(field);
            setSortOrder('asc');
        }
    };

    return (
        <div>
            <table>
                <thead>
                    <tr>
                        <th onClick={() => toggleSort('name')}>
                            Name {sortBy === 'name' && (sortOrder === 'asc' ? '↑' : '↓')}
                        </th>
                        <th onClick={() => toggleSort('age')}>
                            Age {sortBy === 'age' && (sortOrder === 'asc' ? '↑' : '↓')}
                        </th>
                    </tr>
                </thead>
                <tbody>
                    {sortedUsers.map(user => (
                        <tr key={user.id}>
                            <td>{user.name}</td>
                            <td>{user.age}</td>
                        </tr>
                    ))}
                </tbody>
            </table>
        </div>
    );
}
```

### Search/Filter Combined

```jsx
function SearchableList() {
    const [search, setSearch] = useState('');
    const [category, setCategory] = useState('all');

    const products = [
        { id: 1, name: 'Laptop', category: 'electronics', price: 999 },
        { id: 2, name: 'T-Shirt', category: 'clothing', price: 29 },
        { id: 3, name: 'Phone', category: 'electronics', price: 699 },
        { id: 4, name: 'Jeans', category: 'clothing', price: 59 },
        { id: 5, name: 'Headphones', category: 'electronics', price: 199 }
    ];

    const filteredProducts = products
        .filter(p => category === 'all' || p.category === category)
        .filter(p => p.name.toLowerCase().includes(search.toLowerCase()));

    return (
        <div>
            <input
                type="search"
                value={search}
                onChange={(e) => setSearch(e.target.value)}
                placeholder="Search products..."
            />

            <select value={category} onChange={(e) => setCategory(e.target.value)}>
                <option value="all">All Categories</option>
                <option value="electronics">Electronics</option>
                <option value="clothing">Clothing</option>
            </select>

            <p>Found {filteredProducts.length} products</p>

            <ul>
                {filteredProducts.map(product => (
                    <li key={product.id}>
                        {product.name} - ${product.price}
                    </li>
                ))}
            </ul>
        </div>
    );
}
```

---

## CRUD Operations on Lists

### Full CRUD Example

```jsx
function TodoApp() {
    const [todos, setTodos] = useState([
        { id: 1, text: 'Learn React', completed: false },
        { id: 2, text: 'Build project', completed: false }
    ]);
    const [newTodo, setNewTodo] = useState('');
    const [editingId, setEditingId] = useState(null);
    const [editText, setEditText] = useState('');

    // CREATE
    const addTodo = (e) => {
        e.preventDefault();
        if (!newTodo.trim()) return;

        const todo = {
            id: Date.now(),
            text: newTodo.trim(),
            completed: false
        };

        setTodos([...todos, todo]);
        setNewTodo('');
    };

    // READ is just rendering the list

    // UPDATE - Toggle completion
    const toggleTodo = (id) => {
        setTodos(todos.map(todo =>
            todo.id === id
                ? { ...todo, completed: !todo.completed }
                : todo
        ));
    };

    // UPDATE - Edit text
    const startEditing = (todo) => {
        setEditingId(todo.id);
        setEditText(todo.text);
    };

    const saveEdit = (id) => {
        if (!editText.trim()) return;

        setTodos(todos.map(todo =>
            todo.id === id
                ? { ...todo, text: editText.trim() }
                : todo
        ));
        setEditingId(null);
        setEditText('');
    };

    const cancelEdit = () => {
        setEditingId(null);
        setEditText('');
    };

    // DELETE
    const deleteTodo = (id) => {
        setTodos(todos.filter(todo => todo.id !== id));
    };

    return (
        <div style={{ maxWidth: '500px', margin: '0 auto' }}>
            <h1>Todo List</h1>

            <form onSubmit={addTodo}>
                <input
                    type="text"
                    value={newTodo}
                    onChange={(e) => setNewTodo(e.target.value)}
                    placeholder="Add new todo..."
                />
                <button type="submit">Add</button>
            </form>

            <ul style={{ listStyle: 'none', padding: 0 }}>
                {todos.map(todo => (
                    <li
                        key={todo.id}
                        style={{
                            display: 'flex',
                            alignItems: 'center',
                            padding: '10px',
                            borderBottom: '1px solid #eee'
                        }}
                    >
                        <input
                            type="checkbox"
                            checked={todo.completed}
                            onChange={() => toggleTodo(todo.id)}
                        />

                        {editingId === todo.id ? (
                            <>
                                <input
                                    type="text"
                                    value={editText}
                                    onChange={(e) => setEditText(e.target.value)}
                                    style={{ flex: 1, margin: '0 10px' }}
                                />
                                <button onClick={() => saveEdit(todo.id)}>Save</button>
                                <button onClick={cancelEdit}>Cancel</button>
                            </>
                        ) : (
                            <>
                                <span style={{
                                    flex: 1,
                                    marginLeft: '10px',
                                    textDecoration: todo.completed ? 'line-through' : 'none',
                                    color: todo.completed ? '#888' : 'inherit'
                                }}>
                                    {todo.text}
                                </span>
                                <button onClick={() => startEditing(todo)}>Edit</button>
                                <button onClick={() => deleteTodo(todo.id)}>Delete</button>
                            </>
                        )}
                    </li>
                ))}
            </ul>

            {todos.length === 0 && (
                <p style={{ textAlign: 'center', color: '#888' }}>
                    No todos yet. Add one above!
                </p>
            )}
        </div>
    );
}
```

---

## Nested Lists

```jsx
function NestedList() {
    const categories = [
        {
            id: 1,
            name: 'Electronics',
            products: [
                { id: 101, name: 'Laptop' },
                { id: 102, name: 'Phone' },
                { id: 103, name: 'Tablet' }
            ]
        },
        {
            id: 2,
            name: 'Clothing',
            products: [
                { id: 201, name: 'T-Shirt' },
                { id: 202, name: 'Jeans' }
            ]
        },
        {
            id: 3,
            name: 'Books',
            products: [
                { id: 301, name: 'Fiction' },
                { id: 302, name: 'Non-Fiction' },
                { id: 303, name: 'Comics' }
            ]
        }
    ];

    return (
        <div>
            {categories.map(category => (
                <div key={category.id}>
                    <h2>{category.name}</h2>
                    <ul>
                        {category.products.map(product => (
                            <li key={product.id}>{product.name}</li>
                        ))}
                    </ul>
                </div>
            ))}
        </div>
    );
}
```

---

## Virtualized Lists (Performance)

For very long lists, consider virtualization:

```jsx
// Using react-window (install: npm install react-window)
import { FixedSizeList } from 'react-window';

function VirtualizedList({ items }) {
    const Row = ({ index, style }) => (
        <div style={style}>
            {items[index].name}
        </div>
    );

    return (
        <FixedSizeList
            height={400}
            width={300}
            itemCount={items.length}
            itemSize={35}
        >
            {Row}
        </FixedSizeList>
    );
}

// Only renders visible items + buffer
// Great for lists with 1000+ items
```

---

## Practice Exercises

### Exercise 1: Reorderable List

```jsx
// Create a list that can be reordered with up/down buttons
// Your code here
```

**Solution:**
```jsx
function ReorderableList() {
    const [items, setItems] = useState([
        { id: 1, name: 'First Item' },
        { id: 2, name: 'Second Item' },
        { id: 3, name: 'Third Item' },
        { id: 4, name: 'Fourth Item' }
    ]);

    const moveUp = (index) => {
        if (index === 0) return;
        const newItems = [...items];
        [newItems[index - 1], newItems[index]] = [newItems[index], newItems[index - 1]];
        setItems(newItems);
    };

    const moveDown = (index) => {
        if (index === items.length - 1) return;
        const newItems = [...items];
        [newItems[index], newItems[index + 1]] = [newItems[index + 1], newItems[index]];
        setItems(newItems);
    };

    return (
        <ul style={{ listStyle: 'none', padding: 0 }}>
            {items.map((item, index) => (
                <li
                    key={item.id}
                    style={{
                        display: 'flex',
                        alignItems: 'center',
                        padding: '10px',
                        marginBottom: '5px',
                        backgroundColor: '#f5f5f5',
                        borderRadius: '4px'
                    }}
                >
                    <span style={{ flex: 1 }}>{item.name}</span>
                    <button
                        onClick={() => moveUp(index)}
                        disabled={index === 0}
                    >
                        ↑
                    </button>
                    <button
                        onClick={() => moveDown(index)}
                        disabled={index === items.length - 1}
                    >
                        ↓
                    </button>
                </li>
            ))}
        </ul>
    );
}
```

### Exercise 2: Grouped List

```jsx
// Create a list grouped by category with collapsible sections
// Your code here
```

**Solution:**
```jsx
function GroupedList() {
    const [expandedGroups, setExpandedGroups] = useState(['electronics']);

    const items = [
        { id: 1, name: 'Laptop', category: 'electronics' },
        { id: 2, name: 'Phone', category: 'electronics' },
        { id: 3, name: 'T-Shirt', category: 'clothing' },
        { id: 4, name: 'Jeans', category: 'clothing' },
        { id: 5, name: 'Apple', category: 'food' },
        { id: 6, name: 'Bread', category: 'food' }
    ];

    // Group items by category
    const grouped = items.reduce((acc, item) => {
        if (!acc[item.category]) {
            acc[item.category] = [];
        }
        acc[item.category].push(item);
        return acc;
    }, {});

    const toggleGroup = (category) => {
        setExpandedGroups(prev =>
            prev.includes(category)
                ? prev.filter(c => c !== category)
                : [...prev, category]
        );
    };

    return (
        <div>
            {Object.entries(grouped).map(([category, categoryItems]) => (
                <div key={category} style={{ marginBottom: '10px' }}>
                    <button
                        onClick={() => toggleGroup(category)}
                        style={{
                            width: '100%',
                            padding: '10px',
                            textAlign: 'left',
                            backgroundColor: '#333',
                            color: 'white',
                            border: 'none',
                            cursor: 'pointer',
                            display: 'flex',
                            justifyContent: 'space-between'
                        }}
                    >
                        <span style={{ textTransform: 'capitalize' }}>
                            {category} ({categoryItems.length})
                        </span>
                        <span>{expandedGroups.includes(category) ? '−' : '+'}</span>
                    </button>

                    {expandedGroups.includes(category) && (
                        <ul style={{
                            listStyle: 'none',
                            padding: 0,
                            margin: 0,
                            backgroundColor: '#f5f5f5'
                        }}>
                            {categoryItems.map(item => (
                                <li
                                    key={item.id}
                                    style={{ padding: '10px', borderBottom: '1px solid #ddd' }}
                                >
                                    {item.name}
                                </li>
                            ))}
                        </ul>
                    )}
                </div>
            ))}
        </div>
    );
}
```

---

## Key Takeaways

1. **Use map()** - Transform arrays into JSX elements
2. **Always use keys** - Help React identify items efficiently
3. **Use stable, unique IDs** - Not array indexes for dynamic lists
4. **Keys on array elements** - Not inside child components
5. **Filter then map** - For conditional list rendering
6. **Don't mutate** - Create new arrays when updating
7. **Consider virtualization** - For very long lists

---

## Self-Check Questions

1. Why are keys important in React lists?
2. When is it okay to use array index as key?
3. Where should you put the key prop?
4. How do you filter a list in React?
5. How do you update an item in a list state?
6. What is list virtualization?

---

**Next Lesson:** [Day 7 - Forms](./day-07-forms.md)
