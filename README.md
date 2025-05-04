# PrismaJSON

A database layer for Electron apps using a local JSON file with a Prisma-like API. PrismaJSON allows you to work with local JSON files as if they were databases, providing a familiar interface for anyone who has used Prisma ORM.

## Features

- 🔒 **Secure Storage**: AES-256 encryption for data files
- 🧩 **Prisma-like API**: Familiar methods like `create`, `findMany`, `update`, etc.
- 📊 **Schema Support**: Define models and field types in a JSON schema
- 🔍 **Filtering & Sorting**: Filter and sort data with a flexible query API
- 📱 **Offline-First**: No network connection required
- 🌐 **Electron Optimized**: Designed for Electron apps with local storage
- 💻 **TypeScript Support**: Fully typed API and schema

## Installation

```bash
npm install prismajson
```

## Quick Start

### 1. Define a Schema

Create a `schema.json` file:

```json
{
  "models": {
    "User": {
      "fields": {
        "id": {
          "type": "string",
          "isId": true,
          "isRequired": true
        },
        "email": {
          "type": "string",
          "isRequired": true,
          "isUnique": true
        },
        "name": {
          "type": "string",
          "isRequired": true
        },
        "age": {
          "type": "number"
        }
      }
    },
    "Post": {
      "fields": {
        "id": {
          "type": "string",
          "isId": true,
          "isRequired": true
        },
        "title": {
          "type": "string",
          "isRequired": true
        },
        "content": {
          "type": "string",
          "isRequired": true
        },
        "authorId": {
          "type": "string",
          "isRequired": true,
          "ref": "User"
        }
      }
    }
  }
}
```

### 2. Initialize the Client

```typescript
import { createPrismaJsonClient } from 'prismajson';
import * as path from 'path';
import * as os from 'os';

const client = createPrismaJsonClient({
  schemaPath: path.join(__dirname, 'schema.json'),
  dataDir: path.join(os.homedir(), '.myapp-data'),
  encryptData: true,
});
```

### 3. Use the Client

```typescript
// Create a user
const user = await client.create('User', {
  email: 'john@example.com',
  name: 'John Doe',
  age: 30,
});

// Create a post
const post = await client.create('Post', {
  title: 'Hello World',
  content: 'This is my first post',
  authorId: user.id,
});

// Find all posts by a user
const userPosts = await client.findMany('Post', {
  where: {
    authorId: user.id,
  },
});

// Update a post
const updatedPost = await client.update('Post', {
  where: { id: post.id },
  data: {
    title: 'Updated Title',
  },
});

// Delete a post
const deletedPost = await client.delete('Post', {
  id: post.id,
});
```

## API Reference

### Client Initialization

```typescript
createPrismaJsonClient(options: PrismaJsonOptions): PrismaJsonClient
```

Options:
- `schemaPath` (required): Path to the schema JSON file
- `dataDir` (optional): Directory where data files will be stored (defaults to `~/.prismajson`)
- `encryptionKey` (optional): Custom encryption key for AES-256 encryption
- `encryptData` (optional): Whether to encrypt data (defaults to `true`)

### CRUD Operations

#### Create

```typescript
create(modelName: string, data: Record<string, any>): Promise<DataRecord>
```

#### Find Many

```typescript
findMany(modelName: string, options?: {
  where?: FilterCondition,
  orderBy?: Record<string, SortOrder>,
  take?: number,
  skip?: number,
}): Promise<DataRecord[]>
```

#### Find Unique

```typescript
findUnique(modelName: string, where: { id: string }): Promise<DataRecord | null>
```

#### Find First

```typescript
findFirst(modelName: string, options?: {
  where?: FilterCondition,
  orderBy?: Record<string, SortOrder>,
}): Promise<DataRecord | null>
```

#### Update

```typescript
update(modelName: string, args: {
  where: { id: string },
  data: Record<string, any>,
}): Promise<DataRecord>
```

#### Update Many

```typescript
updateMany(modelName: string, args: {
  where: FilterCondition,
  data: Record<string, any>,
}): Promise<{ count: number }>
```

#### Delete

```typescript
delete(modelName: string, where: { id: string }): Promise<DataRecord>
```

#### Delete Many

```typescript
deleteMany(modelName: string, where: FilterCondition): Promise<{ count: number }>
```

#### Count

```typescript
count(modelName: string, options?: {
  where?: FilterCondition,
}): Promise<number>
```

### Query Filters

You can use the following operators in your `where` conditions:

- `equals`: Exact match
- `not`: Not equal
- `in`: Value in array
- `notIn`: Value not in array
- `lt`: Less than
- `lte`: Less than or equal
- `gt`: Greater than
- `gte`: Greater than or equal
- `contains`: String contains
- `startsWith`: String starts with
- `endsWith`: String ends with
- `AND`: Logical AND of conditions
- `OR`: Logical OR of conditions
- `NOT`: Logical NOT of a condition

Example:

```typescript
const results = await client.findMany('User', {
  where: {
    OR: [
      { name: { contains: 'John' } },
      { email: { endsWith: '@example.com' } }
    ],
    age: { gte: 18 },
    NOT: { name: 'Admin' }
  },
  orderBy: { age: 'desc' },
  take: 10,
  skip: 0
});
```

## Schema Definition

### Field Types

- `string`: String values
- `number`: Numeric values
- `boolean`: Boolean values
- `date`: Date values (stored as ISO strings)
- `object`: Nested objects with properties
- `array`: Arrays of values

### Field Properties

- `type` (required): The field type
- `isId`: Whether this field is the ID field
- `isRequired`: Whether this field is required
- `isUnique`: Whether this field must have unique values
- `default`: Default value for the field
- `ref`: For references to other models
- `items`: For array items (when type is 'array')
- `properties`: For object properties (when type is 'object')

## Security

PrismaJSON uses AES-256 encryption to secure your data. By default, a random encryption key is generated and stored in your data directory. You can also provide your own encryption key.

## Electron Integration

PrismaJSON is designed to work well with Electron apps. Here's how to integrate it:

```typescript
// In your main process
import { createPrismaJsonClient } from 'prismajson';
import * as path from 'path';
import * as os from 'os';
import { app } from 'electron';

// Use app.getPath('userData') for Electron's user data directory
const client = createPrismaJsonClient({
  schemaPath: path.join(__dirname, 'schema.json'),
  dataDir: path.join(app.getPath('userData'), 'database'),
  encryptData: true,
});

// Expose methods to renderer process via IPC
ipcMain.handle('db:create', async (event, modelName, data) => {
  return client.create(modelName, data);
});

// Add other IPC handlers for other operations
```

## License

MIT
