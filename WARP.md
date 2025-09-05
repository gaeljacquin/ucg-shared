# WARP.md

This file provides guidance to WARP (warp.dev) when working with the shared package.

## Package Overview

**`@gaeljacquin/ucg-shared`** is a TypeScript library that provides core game logic, card definitions, and shared types for the Untitled Card Game. It's published to NPM and consumed by both the client (Next.js) and server (NestJS) applications.

## Development Commands

### Basic Workflow
```bash
pnpm install              # Install dependencies  
pnpm build               # Compile TypeScript to dist/
pnpm prettier:write      # Format code
```

### Development & Testing
```bash
# Build and check output
pnpm build && ls dist/

# Format code before commits
pnpm prettier:write

# Manual NPM publish (usually done via GitHub Actions)
npm publish --access public
```

## Project Structure

```
shared/
├── src/
│   ├── core/              # Core game classes
│   │   ├── card.ts        # ABCard class
│   │   ├── deck.ts        # ABDeck class  
│   │   ├── game.ts        # ABGame class
│   │   ├── mode.ts        # ABMode class
│   │   ├── grid-cell.ts   # IABGridCell interface
│   │   ├── poker-hand.ts  # Hand evaluation
│   │   ├── rank.ts        # Card rank definitions
│   │   └── suit.ts        # Card suit definitions
│   ├── constants/         # Static data
│   │   ├── empty-hand.ts  # Default hand state
│   │   └── suit-icon.ts   # Suit icon mappings  
│   ├── functions/         # Utility functions
│   │   ├── evaluate.ts    # Game scoring logic
│   │   └── shuffle.ts     # Deck shuffling
│   └── index.ts          # Package exports
├── dist/                 # Compiled JavaScript (generated)
├── package.json          # Package configuration
└── tsconfig.json         # TypeScript configuration
```

## Key Exports

### Core Classes
- **`ABCard`** - Individual playing card with rank, suit, and state
- **`ABDeck`** - Deck management and card dealing
- **`ABGame`** - Game state and round management  
- **`ABMode`** - Game mode definitions and configurations
- **`ABGridCell`** - Grid cell interface for game board

### Game Logic Functions
- **`calculateScore()`** - Scoring calculation for completed grids
- **`evaluateGridRow()`** - Evaluate poker hand in a row
- **`evaluateGridColumn()`** - Evaluate poker hand in a column

### Type Definitions
- **`ABCards`** - Array of ABCard objects
- **`IABGridCell`** - Grid cell interface
- **`SlugId`** - String literal type for game modes
- **`IABModeType`** - Game mode type

### Constants
- **`emptyHand`** - Default empty hand state
- **`suitIconMap`** - Mapping of suit IDs to React icon components

## TypeScript Configuration

### Build Settings
- **Target**: ES6
- **Module**: CommonJS  
- **Output**: `./dist/` directory
- **Declarations**: Generated for TypeScript consumers
- **Strict Mode**: Enabled

### Path Aliases
- **`@/*`** → `./src/*` (for internal imports)

## Publishing Workflow

### Automated Publishing (Preferred)
1. Make changes in `/shared/src/`
2. Commit and push to feature branch
3. Create PR to `main` branch
4. On PR merge, GitHub Actions automatically:
   - Builds the package
   - Publishes to NPM with version from `package.json`

### Manual Publishing (Emergency Only)
```bash
# Ensure clean build
pnpm build

# Verify package contents
npm pack
tar -tzf gaeljacquin-ucg-shared-*.tgz

# Publish to NPM
npm publish --access public
```

## Development Notes

### Version Management
- Update `package.json` version before publishing
- Follow semantic versioning (semver)
- Coordinate with main app developers for version updates

### Breaking Changes
When making breaking changes:
1. Update version appropriately (major version bump)
2. Test with main application in `/source/`
3. Update client and server `package.json` files
4. Document changes for application developers

### Testing Strategy
- The shared package has no direct tests
- Validation occurs through integration with client/server
- Use the main app's smoke test: `npx -y -p socket.io-client -p tsx tsx scripts/ws-smoke-test.mts` (run from `/source/`)

### Dependencies
- **Runtime**: `react-icons` (for suit icon components)
- **Dev**: `typescript` (for compilation)
- **Minimal footprint**: Keep dependencies light for consuming applications

## Integration with Main Application

### Client Usage
```typescript
import { ABMode, ABCard, calculateScore } from '@gaeljacquin/ucg-shared';

const mode = ABMode.getMode('classic');
const scores = calculateScore(grid, mode, discardPile);
```

### Server Usage  
```typescript
import { ABDeck, ABGame, ABMode } from '@gaeljacquin/ucg-shared';

const deck = new ABDeck();
const game = new ABGame(mode);
const cards = game.dealHand(roundIndex);
```

### Local Development Linking
For testing unpublished changes:
```bash
# In /shared directory
npm link

# In /source/client or /source/server
npm link @gaeljacquin/ucg-shared

# Remember to unlink and reinstall when done
npm unlink @gaeljacquin/ucg-shared
pnpm install
```

## Common Development Tasks

### Adding New Game Logic
1. Add functionality to appropriate file in `/src/core/` or `/src/functions/`
2. Export from `/src/index.ts`
3. Build and test with main application
4. Update version and publish

### Modifying Card/Game Types  
1. Update interfaces in `/src/core/`
2. Ensure backward compatibility or coordinate breaking changes
3. Test thoroughly with both client and server

### Adding New Constants
1. Add to appropriate file in `/src/constants/`
2. Export from `/src/index.ts`
3. Update consuming applications if needed
