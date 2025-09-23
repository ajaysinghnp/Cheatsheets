# Next.js Project Setup Guide

This guide will walk you through setting up a modern Next.js project with ESLint, Prettier, and other essential development tools.

## Prerequisites

- Node.js 18.17 or later
- npm, yarn, or pnpm

## Initial Setup

### 1. Create Next.js Project

Run the following command and answer the prompts interactively:

```bash
npx create-next-app@latest my-nextjs-app
```

When prompted, select the following options:

- **Would you like to use TypeScript?** → Yes
- **Would you like to use ESLint?** → Yes
- **Would you like to use Tailwind CSS?** → Yes
- **Would you like to use `src/` directory?** → No
- **Would you like to use App Router?** → Yes
- **Would you like to customize the default import alias?** → No

Then navigate to the project:

```bash
cd my-nextjs-app
```

### 2. Install Development Dependencies

```bash
# using npm

npm install @eslint/js

npm install -D prettier eslint-config-prettier eslint-plugin-prettier husky lint-staged @t3-oss/env-nextjs zod @trivago/prettier-plugin-sort-imports prettier-plugin-tailwindcss eslint-plugin-check-file eslint-plugin-n
```

```bash
# using pnpm

pnpm install @eslint/js

pnpm add -D prettier eslint-config-prettier eslint-plugin-prettier husky lint-staged @t3-oss/env-nextjs zod @trivago/prettier-plugin-sort-imports prettier-plugin-tailwindcss eslint-plugin-check-file eslint-plugin-n
```

```bash
# using yarn

yarn install @eslint/js

yarn add -D prettier eslint-config-prettier eslint-plugin-prettier husky lint-staged @t3-oss/env-nextjs zod @trivago/prettier-plugin-sort-imports prettier-plugin-tailwindcss eslint-plugin-check-file eslint-plugin-n
```

## Configuration Files

### 3. ESLint Configuration

1. Using `.mjs` file (`.eslintrc..mjs`)

```typescript
import { FlatCompat } from "@eslint/eslintrc";

import checkFilePlugin from "eslint-plugin-check-file";
import prettierPlugin from "eslint-plugin-prettier";
import { dirname } from "path";
import { fileURLToPath } from "url";

const __filename = fileURLToPath(import.meta.url);
const__dirname = dirname(__filename);

const compat = new FlatCompat({
  baseDirectory: __dirname,
});

/** @type {import('eslint').Linter.Config[]} */
const eslintConfig = [
  ...compat.extends(
    "next/core-web-vitals",
    "next/typescript",
    "prettier",
    "plugin:n/recommended"
  ),
  {
    ignores: [
      "node_modules/**",
      ".next/**",
      "out/**",
      "build/**",
      "next-env.d.ts",
    ],
  },
  {
    plugins: {
      prettier: prettierPlugin,
      "check-file": checkFilePlugin,
    },
    rules: {
      "prettier/prettier": "error",
      "react-hooks/exhaustive-deps": "off",
      "n/no-missing-import": "off",
      "n/no-unpublished-import": "off",
      "check-file/filename-naming-convention": [
        "error",
        {
          "**/*.{js,jsx,ts,tsx}": "KEBAB_CASE",
        },
        {
          ignoreMiddleExtensions: true,
        },
      ],
      "check-file/folder-naming-convention": [
        "error",
        {
          "app/**/": "NEXT_JS_APP_ROUTER_CASE",
          "components/**/": "NEXT_JS_APP_ROUTER_CASE",
          "lib/**/": "NEXT_JS_APP_ROUTER_CASE",
          "hooks/**/": "NEXT_JS_APP_ROUTER_CASE",
        },
      ],
    },
  },
];

export default eslintConfig;
```

2. Using `json` file (`.eslintrc.json`)

Update or create the ESLint configuration:

```json
{
  "extends": [
    "next/core-web-vitals",
    "prettier",
    "plugin:n/recommended"
  ],
  "plugins": [
    "prettier",
    "check-file"
  ],
  "rules": {
    "prettier/prettier": "error",
    "react-hooks/exhaustive-deps": "off",
    "n/no-missing-import": "off",
    "n/no-unpublished-import": "off",
    "check-file/filename-naming-convention": [
      "error",
      {
        "**/*.{js,jsx,ts,tsx}": "KEBAB_CASE"
      }
    ],
    "check-file/folder-naming-convention": [
      "error",
      {
        "app/**/": "KEBAB_CASE",
        "components/**/": "KEBAB_CASE",
        "lib/**/": "KEBAB_CASE",
        "hooks/**/": "KEBAB_CASE"
      }
    ]
  }
}
```

### 4. Prettier Configuration (`.prettierrc`)

Create a Prettier configuration file:

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": false,
  "tabWidth": 2,
  "useTabs": false,
  "printWidth": 80,
  "endOfLine": "lf",
  "plugins": [
    "@trivago/prettier-plugin-sort-imports",
    "prettier-plugin-tailwindcss"
  ],
  "importOrder": [
    "^react",
    "^next",
    "<THIRD_PARTY_MODULES>",
    "^@/(.*)$",
    "^[./]"
  ],
  "importOrderSeparation": true,
  "importOrderSortSpecifiers": true
}
```

### 5. Prettier Ignore (`.prettierignore`)

Create a Prettier ignore file:

```ignore
.next
node_modules
public
*.md
*.json
```

### 6. TypeScript Configuration (`tsconfig.json`)

Update your TypeScript configuration:

```json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "es6"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "baseUrl": ".",
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### 7. Type-Safe Environment Variables

Create `lib/env.js`:

```javascript
import { createEnv } from '@t3-oss/env-nextjs';
import { z } from 'zod';

export const env = createEnv({
  server: {
    NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
    DATABASE_URL: z.string().url(),
  },
  client: {
    NEXT_PUBLIC_APP_URL: z.string().url(),
    NEXT_PUBLIC_API_URL: z.string().url(),
  },
  runtimeEnv: {
    NODE_ENV: process.env.NODE_ENV,
    DATABASE_URL: process.env.DATABASE_URL,
    NEXT_PUBLIC_APP_URL: process.env.NEXT_PUBLIC_APP_URL,
    NEXT_PUBLIC_API_URL: process.env.NEXT_PUBLIC_API_URL,
  },
});
```

Create `.env.example`:

```env
# Environment
NODE_ENV=development

# Database
DATABASE_URL=your_database_url_here

# Public (client-side) environment variables
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_API_URL=http://localhost:3000/api
```

Update your `.env.local` with actual values.

### 8. Git Hooks Setup with Husky

Initialize Husky and set up git hooks:

```bash
# using npm

npx husky init
npm pkg set scripts.prepare="husky install"
```

```bash
# using pnpm

npx husky init
pnpm pkg set scripts.prepare="husky install"
```

```bash
# using yarn

npx husky init
yarn pkg set scripts.prepare="husky install"
```

Add pre-commit hook:

```bash
npx husky add .husky/pre-commit "npx lint-staged"
```

sample `pre-commit` file:

```bash
#!/usr/bin/env sh
echo "🏃 Running pre-commit checks..."

# Run lint-staged
pnpm exec lint-staged

# Check if lint-staged passed
if [ $? -ne 0 ]; then
    echo "❌ Pre-commit checks failed. Please fix the errors and try again."
    exit 1
fi

# Generate .sample.env file from .env
npx helper gen-env --silent --name .env --sample .sample.env

# Check if sample .env file generated successfully
if [ $? -ne 0 ]; then
    echo "❌ Sample .env file generation failed. Please fix the errors and try again."
    exit 1
fi

git add .

echo "✅ Pre-commit checks passed!"
```

### 9. Lint-Staged Configuration

Add to your `package.json`:

```json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,md,mdx,css,html,yml,yaml}": [
      "prettier --write"
    ]
  }
}
```

## Project Structure

Your project structure should look like this (without `src/` directory):

```bash
my-nextjs-app/
├── app/                    # App Router (Next.js 13+)
├── components/             # Reusable components
├── lib/                   # Utility functions and configurations
├── hooks/                 # Custom React hooks
├── types/                 # TypeScript type definitions
├── public/                # Static assets
├── .eslintrc.json         # ESLint configuration
├── .prettierrc            # Prettier configuration
├── .prettierignore        # Prettier ignore patterns
├── tsconfig.json          # TypeScript configuration
└── package.json
```

## Scripts Configuration

Update your `package.json` scripts:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "lint:fix": "next lint --fix",
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "type-check": "tsc --noEmit",
    "env:check": "tsc --noEmit && next build",
    "prepare": "husky install"
  }
}
```

## VS Code Configuration (Optional)

Create `.vscode/settings.json` for better developer experience:

```json
{
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  },
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "typescript.preferences.importModuleSpecifier": "non-relative",
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

Create `.vscode/extensions.json` for recommended extensions:

```json
{
  "recommendations": [
    "bradlc.vscode-tailwindcss",
    "esbenp.prettier-vscode",
    "dbaeumer.vscode-eslint"
  ]
}
```

## Final Steps

### 10. Run Initial Setup

Using npm

```bash
# Install dependencies
npm install

# Run type checking
npm run type-check

# Run linting
npm run lint

# Format code
npm run format

# Check environment variables
npm run env:check

# Test the build
npm run build
```

Using pnpm

```bash
# Install dependencies
pnpm install

# Run type checking
pnpm type-check

# Run linting
pnpm lint

# Format code
pnpm format

# Check environment variables
pnpm env:check

# Test the build
pnpm build
```

Using yarn

```bash
# Install dependencies
yarn install

# Run type checking
yarn type-check

# Run linting
yarn lint

# Format code
yarn format

# Check environment variables
yarn env:check

# Test the build
yarn build
```

### 11. Verify Setup

1. Start development server:

Using npm

```bash
npm run dev
```

Using pnpm

```bash
pnpm dev
```

Using yarn

```bash
yarn dev
```

1. Visit `http://localhost:3000`
2. Check that there are no ESLint errors in the console
3. Verify Prettier formatting works by saving a file
4. Verify import sorting is working correctly

## Usage

### Development

Using npm

```bash
npm run dev
```

Using pnpm

```bash
pnpm dev
```

Using yarn

```bash
yarn dev
```

### Linting and Formatting

Using npm

```bash
# Fix lint issues
npm run lint:fix

# Format code
npm run format

# Check formatting without applying
npm run format:check
```

Using pnpm

```bash
# Fix lint issues
pnpm lint:fix

# Format code
pnpm format

# Check formatting without applying
pnpm format:check
```

Using yarn

```bash
# Fix lint issues
yarn lint:fix

# Format code
yarn format

# Check formatting without applying
yarn format:check
```

### Production Build

Using npm

```bash
npm run build
npm start
```

Using pnpm

```bash
pnpm build
pnpm start
```

Using yarn

```bash
yarn build
yarn start
```

## Environment Variables Management

Always use the type-safe environment variables:

```typescript
import { env } from '@/lib/env';

// Server component
export function ServerComponent() {
  console.log(env.DATABASE_URL); // Type-safe server env
}

// Client component
'use client';
export function ClientComponent() {
  return <div>API URL: {env.NEXT_PUBLIC_API_URL}</div>; // Type-safe client env
}
```

## Git Ignore

Ensure your `.gitignore` includes:

```ignore
# Dependencies
/node_modules
/.pnp
.pnp.js

# Next.js
/.next/
/out/

# Production
/build

# Misc
.DS_Store
*.pem

# Debug
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*

# Local env files
.env*.local
.env
!.env.example

# IDE
.vscode/
.idea/

# Husky
.husky/
```

This setup provides a comprehensive foundation for a Next.js project with proper linting, formatting, type-safe environment variables, and modern development tools configured.
