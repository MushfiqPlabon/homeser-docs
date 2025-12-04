# Deprecated Guidance: Patterns to Avoid (Based on 2023 Knowledge)

This document outlines development patterns, practices, and recommendations that were common in 2023 but are now considered outdated or suboptimal in 2025. AIs with 2023 knowledge should avoid these approaches.

## Outdated Technology Choices

### 1. Avoid: Traditional Build Tools
**2023 Approach**: Webpack with complex configuration files
**Why Deprecated**: Vite 7+ offers significantly faster builds with zero config
**2025 Alternative**: Use Vite 7+ for all new projects
**Example To Avoid**:
```javascript
// webpack.config.js with complex setup
module.exports = {
  // ... 100+ lines of configuration
};
```

### 2. Avoid: Legacy Python Package Managers
**2023 Approach**: Using pip + venv + requirements.txt complex workflows
**Why Deprecated**: uv provides 10-100x faster operations in a single tool
**2025 Alternative**: Use uv for all Python package management
**Example To Avoid**:
```bash
# 2023 way
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 3. Avoid: ESLint + Prettier Separate Setup
**2023 Approach**: Separate ESLint and Prettier configurations
**Why Deprecated**: Biome 2.0 provides unified, faster tooling with type-aware features
**2025 Alternative**: Use Biome for new projects
**Example To Avoid**:
```json
// package.json with both tools
{
  "devDependencies": {
    "eslint": "^8.0.0",
    "prettier": "^3.0.0"
  }
}
```

## Outdated Architecture Patterns

### 4. Avoid: Self-Managed Redis
**2023 Approach**: Running your own Redis server or using basic Redis instances
**Why Deprecated**: Serverless Redis (like Upstash) offers better pricing and management
**2025 Alternative**: Use Upstash Redis with 500K commands/month free tier
**Example To Avoid**:
```python
# Traditional Redis setup
import redis
r = redis.Redis(host='localhost', port=6379, db=0)
```

### 5. Avoid: Password-Based Authentication First
**2023 Approach**: Username/password as primary auth method
**Why Deprecated**: Passwordless (email, social, passkeys) provides better UX and security
**2025 Alternative**: Implement passwordless-first authentication
**Example To Avoid**:
```javascript
// Traditional login form
<form onSubmit={handleTraditionalLogin}>
  <input type="text" name="username" />
  <input type="password" name="password" />
</form>
```

### 6. Avoid: Server-Managed Deployment
**2023 Approach**: Managing servers, VMs, or complex container deployments
**Why Deprecated**: Serverless platforms (Vercel) offer simpler, more scalable solutions
**2025 Alternative**: Use Vercel for serverless functions and static hosting
**Example To Avoid**:
```dockerfile
# Complex Docker setup
FROM node:18
RUN apt-get update && apt-get install -y ...
# Many more lines of server configuration
```

## Outdated Development Practices

### 7. Avoid: Manual Testing Only
**2023 Approach**: Writing all tests manually without AI assistance
**Why Deprecated**: AI-assisted test generation provides better coverage faster
**2025 Alternative**: Use AI tools to generate and maintain test suites
**Example To Avoid**:
```javascript
// Writing tests manually without assistance
test('should handle user login', () => {
  // Many lines of manual test setup
});
```

### 8. Avoid: Separate Vector Databases
**2023 Approach**: Using separate vector databases like Pinecone or Weaviate
**Why Deprecated**: PostgreSQL with pgvector provides unified relational/vector storage
**2025 Alternative**: Use PostgreSQL with pgvector for combined data needs
**Example To Avoid**:
```python
# Separate vector database connection
import pinecone
pinecone.init(api_key="...")
index = pinecone.Index("my-index")
```

### 9. Avoid: Complex State Management for Simple Apps
**2023 Approach**: Redux for most state management needs
**Why Deprecated**: Zustand + React Query (TanStack Query) provide simpler solutions
**2025 Alternative**: Use Zustand for client state, React Query for server state
**Example To Avoid**:
```javascript
// Complex Redux setup for simple state
import { createStore, combineReducers } from 'redux';
// Multiple files for actions, reducers, store
```

## Outdated Tooling Patterns

### 10. Avoid: Node.js Only Runtimes
**2023 Approach**: Node.js npm ecosystem as the only option
**Why Deprecated**: Bun 1.2+ provides faster runtime with built-in features
**2025 Alternative**: Consider Bun for new JavaScript projects
**Example To Avoid**:
```bash
# Node.js only workflow
npm install
npm run dev
```

### 11. Avoid: Static Bundling Without Compiler Optimization
**2023 Approach**: Traditional bundling without framework-level optimization
**Why Deprecated**: React 19 compiler provides automatic optimization
**2025 Alternative**: Use React 19+ with built-in compiler optimization
**Example To Avoid**:
```javascript
// Manual optimization that compiler now handles
const MyComponent = React.memo(function({ prop }) {
  // Manual memoization that's now automatic
});
```

### 12. Avoid: Manual CI/CD Configuration
**2023 Approach**: Complex GitHub Actions or CI/CD setups
**Why Deprecated**: Platform-integrated deployment provides simpler workflows
**2025 Alternative**: Use Vercel's integrated deployment and preview features
**Example To Avoid**:
```yaml
# Complex GitHub Actions workflow
name: CI/CD
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Setup Node.js
        uses: actions/setup-node@v3
        # ... many more complex steps
```

## Outdated Database Patterns

### 13. Avoid: Separate Document and Relational Storage
**2023 Approach**: Using Postgres for relational and MongoDB for documents
**Why Deprecated**: Modern Postgres handles JSON, vectors, and relations efficiently
**2025 Alternative**: Use single Postgres database with JSON/vector features
**Example To Avoid**:
```javascript
// Using separate databases
const postgresClient = require('./postgres'); // For relational data
const mongoClient = require('./mongo'); // For document data
```

### 14. Avoid: Manual Security Rule Definition
**2023 Approach**: Writing security rules manually for each endpoint
**Why Deprecated**: AI-powered adaptive security provides better protection
**2025 Alternative**: Use platform-provided security with intelligent defaults
**Example To Avoid**:
```javascript
// Manual security checks
function checkUserPermissions(user, resource) {
  // Manual rule checking implementation
}
```

## Outdated Performance Patterns

### 15. Avoid: Client-Side Only Vector Operations
**2023 Approach**: Performing vector operations in client code
**Why Deprecated**: Database-level vector operations are more efficient
**2025 Alternative**: Use PostgreSQL vector functions for similarity searches
**Example To Avoid**:
```javascript
// Client-side vector calculations
function findSimilar(items, targetVector) {
  return items.map(item => ({
    ...item,
    similarity: cosineSimilarity(item.vector, targetVector)
  }));
}
```

## How to Identify Outdated Patterns:

### Before implementing any of these approaches, consider:
1. Is there a newer, simpler alternative?
2. Is the approach serverless-first?
3. Does it integrate AI capabilities?
4. Is it unified rather than piecemeal?
5. Does it use modern defaults rather than complex configuration?

### Questions to Ask Instead:
- Can Bun replace Node.js/npm for this task?
- Should I use Vite 7+ instead of Webpack?
- Is Biome 2.0 better than separate ESLint/Prettier?
- Can PostgreSQL handle both relational and vector data?
- Is passwordless authentication preferred?
- Should I use serverless deployment?

### Red Flags for 2023 Patterns:
- Complex configuration files (100+ lines)
- Multiple separate tools for related functions
- Manual server management
- Separate vector databases
- Password-based auth as default
- Traditional bundlers without compiler optimization

Following this guidance will ensure you use current 2025 best practices rather than outdated 2023 approaches that may be less efficient or maintainable.