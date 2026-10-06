# Universal AI Game Player - Architecture Documentation

## System Overview

The Universal AI Game Player is a modular, extensible system designed to understand and interact with games through their visible user interface. It operates on a pipeline model:

```
Game Screen → Capture → Vision → Understanding → State → Decision → Action → Verification → Repeat
```

## Core Components

### 1. Screen Capture (`src/lib/screen-capture.ts`)

**Purpose**: Capture game screen frames for analysis

**Key Classes**:
- `ScreenCapture` - Manages screen capture and frame extraction

**Key Methods**:
```typescript
initialize(region?: ScreenRegion): Promise<void>    // Start screen capture
captureFrame(region?: ScreenRegion): ImageData | null // Get current frame
stop(): void                                          // Stop capture
isActive(): boolean                                   // Check if active
```

**Features**:
- Supports full-screen and region-based capture
- Uses Web APIs (MediaDevices API)
- Non-blocking frame extraction
- Canvas-based rendering

### 2. Game Adapters (`src/lib/game-adapters/`)

**Purpose**: Specialized game understanding and AI decision-making

**Architecture**:
```
BaseGameAdapter (abstract class)
├── TicTacToeAdapter
├── ChessAdapter (future)
├── LudoAdapter (future)
└── GenericGameAdapter (future)
```

**BaseGameAdapter Interface**:
```typescript
abstract class BaseGameAdapter {
  detectGame(frame): boolean                           // Check if game is detected
  parseState(frame): GameState                         // Extract state from frame
  getLegalActions(state): GameAction[]                 // Get available actions
  chooseAction(state): GameAction                      // Select best action
  executeAction(action): Promise<boolean>              // Perform action
  verifyAction(prev, curr, action): boolean           // Confirm action worked
  isGameOver(state): boolean                           // Check game end
  explainAction(state, action): string                // Human-readable explanation
}
```

### 3. Game State (`src/lib/game-adapters/types.ts`)

**Core Types**:
```typescript
interface GameState {
  gameType: string                    // e.g., "tic-tac-toe"
  objects: GameObject[]               // Detected game objects
  players: Player[]                   // Player information
  currentPlayer: number | null        // Current player index
  availableActions: GameAction[]      // Possible actions
  score: Record<string, number>       // Scores
  timer: number | null               // Time remaining
  status: "playing" | "ended" | ...  // Game status
  confidence: number                 // 0-1 confidence in state
}

interface GameAction {
  type: "click" | "drag" | "type" | ... // Action type
  startPosition?: { x, y }               // Start coordinates
  endPosition?: { x, y }                 // End coordinates
  confidence: number                     // 0-1 confidence
  reasoning: string                      // Why this action
}
```

### 4. AI Coach (`src/lib/ai-coach.ts`)

**Purpose**: Provide recommendations and voice guidance

**Key Methods**:
```typescript
analyzeAndRecommend(state): CoachRecommendation     // Get recommendation
getVoiceInstruction(rec): string                    // Format for speech
speak(text): Promise<void>                          // TTS output
setLanguage(lang): void                             // Set voice language
setTTSEnabled(enabled): void                        // Toggle TTS
```

**Features**:
- Multi-language support (English, Hindi)
- Confidence scoring
- Action reasoning
- Web Speech API integration

### 5. Game State Manager (`src/lib/game-state-manager.ts`)

**Purpose**: Centralized state management for game sessions

**Key Classes**:
- `GameStateManager` - Manages current game state and sessions
- `GameSession` - Represents a play session

**Key Methods**:
```typescript
setState(state): void                               // Update state
getState(): GameState | null                        // Get current state
recordAction(action): void                          // Log action
startSession(game, mode): GameSession              // Begin session
endSession(result?): GameSession | null            // End session
getSessionStats(): object | null                   // Get statistics
```

**Features**:
- State change notifications (observer pattern)
- Action history tracking
- Session lifecycle management
- Statistics calculation

### 6. Game Profile Manager (`src/lib/game-profile-manager.ts`)

**Purpose**: Game configuration and profile management

**Key Methods**:
```typescript
getAllProfiles(): Promise<GameProfile[]>            // List all profiles
getProfile(id): Promise<GameProfile | null>         // Get by ID
createProfile(profile): Promise<GameProfile | null> // Create new
updateProfile(id, updates): Promise<GameProfile>    // Update existing
deleteProfile(id): Promise<boolean>                 // Delete
duplicateProfile(id): Promise<GameProfile | null>   // Clone profile
exportProfile(profile): string                      // Export as JSON
importProfile(json): Promise<GameProfile | null>    // Import from JSON
```

**Features**:
- CRUD operations
- Profile import/export
- Filtering by game/platform
- Duplication

## UI Components

### Dashboard (`src/components/dashboard.tsx`)

**Purpose**: Main application interface

**Features**:
- Game selection
- Game connection
- Mode selection (Coach/Auto Play)
- Live preview
- Control buttons
- Status display

**State Management**:
- Mode: "none" | "coach" | "auto_play"
- Connected: boolean
- Selected game: string
- Running: boolean

### Coach Panel (`src/components/coach-panel.tsx`)

**Purpose**: AI Coach mode UI

**Features**:
- Recommendation display
- Confidence visualization
- Voice controls
- Action buttons (Speak, Done, Skip)
- Game status monitoring
- Real-time updates

**Behavior**:
```
Analyze Game State
    ↓
Get AI Recommendation
    ↓
Display Recommendation
    ↓
Speak (optional)
    ↓
Wait for User Action
    ↓
Repeat
```

### Auto Play Panel (`src/components/auto-play-panel.tsx`)

**Purpose**: Automatic gameplay UI

**Features**:
- Action execution display
- Success/failure tracking
- Confidence thresholds
- Action history log
- Statistics
- Pause/resume controls

**Behavior**:
```
Analyze Game State
    ↓
Calculate Best Action
    ↓
Check Confidence
    ↓
Execute Action
    ↓
Verify Result
    ↓
Log Action
    ↓
Repeat
```

## Backend API

### Game Profiles (`/api/game-profiles`)

```
GET  /api/game-profiles           → List profiles
POST /api/game-profiles           → Create profile
GET  /api/game-profiles/[id]      → Get specific profile
PUT  /api/game-profiles/[id]      → Update profile
DEL  /api/game-profiles/[id]      → Delete profile
```

### Game Sessions (`/api/game-sessions`)

```
GET  /api/game-sessions           → List sessions
POST /api/game-sessions           → Create session
```

### Health (`/api/health`)

```
GET  /api/health                  → Check API health
```

## Database Schema

### gameProfiles
Stores game configurations and learned information

```
id                 - Primary key
name              - Profile name
gameName          - Game identifier
platform          - Platform (browser, windows, android)
adapter           - Adapter type (chess, tic-tac-toe, generic)
description       - Human description
rules             - JSON: Learned game rules
controls          - JSON: Control mappings
objects           - JSON: Object definitions
actions           - JSON: Action definitions
winCondition      - JSON: Win/lose/draw conditions
confidenceThreshold - Min confidence (0-1)
isBuiltIn         - Built-in or user-created
isLearned         - AI-learned or manual
createdAt         - Creation timestamp
updatedAt         - Last update timestamp
```

### gameSessions
Tracks game play sessions

```
id                 - Primary key
profileId          - Foreign key to gameProfiles
mode              - "coach" or "auto_play"
startTime         - Session start
endTime           - Session end
result            - "win", "loss", "draw", "stopped"
totalActions      - Number of actions
successfulActions - Actions that succeeded
failedActions     - Actions that failed
averageConfidence - Mean confidence score
metadata          - JSON: Extra data
createdAt         - Timestamp
```

### aiActions
Individual actions taken during sessions

```
id                 - Primary key
sessionId          - Foreign key to gameSessions
actionType        - Type of action
description       - Human description
startX, startY    - Starting coordinates
endX, endY        - Ending coordinates
confidence        - Confidence score (0-1)
verified          - Verification result
reasoning         - Why this action was taken
timestamp         - Action timestamp
```

### gameStatistics
Aggregated statistics

```
id                 - Primary key
profileId          - Foreign key to gameProfiles
gamesPlayed       - Total games
gamesWon          - Won count
gamesLost         - Lost count
gamesDrawn        - Draw count
averageDecisionTime - Mean decision time (ms)
averageConfidence - Mean confidence
successfulActions - Successful actions count
failedActions     - Failed actions count
verificationFailures - Verification failures
detectionAccuracy - Detection accuracy (0-1)
lastPlayedAt      - Last play timestamp
createdAt         - Created timestamp
updatedAt         - Updated timestamp
```

## Game Adapters Explained

### TicTacToeAdapter (Phase 3)

**State Representation**:
```typescript
interface TicTacToeState extends GameState {
  board: ("X" | "O" | null)[]    // 9-element array
  currentPlayerSymbol: "X" | "O"  // Whose turn
  winner: "X" | "O" | "draw" | null
}
```

**AI Algorithm**: Minimax
- Searches game tree recursively
- Evaluates all possible moves
- Returns optimal move
- Time complexity: O(9!) worst case
- Practical: O(10-100ms) for single decision

**Detection**:
1. Capture screen
2. Look for 3x3 grid
3. Detect X/O positions
4. Extract board state

**Verification**:
- Exactly one cell changes
- Cell becomes player's symbol
- No illegal moves

### ChessAdapter (Phase 4 - Planned)

**Features**:
- Visual board recognition
- Piece type identification
- Legal move generation
- Stockfish integration
- FEN position representation

### LudoAdapter (Phase 7 - Planned)

**Features**:
- Token detection
- Dice roll detection
- Board position tracking
- Probability calculation
- Risk assessment

### CarromAdapter (Phase 8 - Planned)

**Features**:
- Board boundary detection
- Striker position
- Coin detection
- Physics simulation
- Angle/power calculation

### GenericGameAdapter (Phase 9 - Planned)

**For unknown games**:
- OCR text detection
- Interactive region identification
- Button detection
- Action discovery
- State change inference

## Data Flow

### Coach Mode Flow

```
┌─────────────────────────────────────┐
│ 1. Capture Screen                   │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│ 2. Parse Game State (Adapter)       │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│ 3. Get Legal Actions (Adapter)      │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│ 4. Choose Best Action (Adapter)     │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│ 5. Explain Action (AICoach)         │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│ 6. Display Recommendation           │
│    - On-screen instruction          │
│    - Confidence score               │
│    - Reasoning                      │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│ 7. Speak Voice Instruction (TTS)    │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│ 8. Wait for User Action             │
└──────────────┬──────────────────────┘
               ↓
           REPEAT
```

### Auto Play Mode Flow

```
┌─────────────────────────────────────┐
│ 1. Capture Screen                   │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│ 2. Parse Game State                 │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│ 3. Get Legal Actions                │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│ 4. Choose Best Action               │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│ 5. Check Confidence                 │
│    >= Threshold?                    │
└──────────────┬──────────────────────┘
           Yes↓ No↓
             │   └──→ PAUSE & WAIT
             ↓
┌─────────────────────────────────────┐
│ 6. Execute Action                   │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│ 7. Wait (configurable delay)        │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│ 8. Capture Next Frame               │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│ 9. Verify Action                    │
│    (Compare states)                 │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│ 10. Log Action Result               │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│ 11. Game Over?                      │
└──────────────┬──────────────────────┘
           Yes↓ No↓
             │   └──→ REPEAT from step 1
             ↓
         END SESSION
```

## Confidence System

Every perception and decision includes a confidence score (0.0 - 1.0):

### Game Detection Confidence
- Board pattern matching
- Piece recognition
- UI element detection

### State Understanding Confidence
- Object position accuracy
- Player turn correctness
- Piece identification

### Action Confidence
- Move legality
- AI evaluation
- Verification success

### Decision Rules

```
Confidence >= 0.8  → Execute with high certainty
0.6 - 0.8         → Execute with caution
< 0.6             → Ask user or pause
```

## Error Handling Strategy

### Graceful Degradation

1. **Detection Fails**
   - Show error message
   - Keep last known state
   - Allow manual correction

2. **State Parse Fails**
   - Try alternate parsing
   - Reduce confidence
   - Request user confirmation

3. **Action Execution Fails**
   - Retry with exponential backoff
   - Log failure
   - Mark as unsuccessful

4. **Verification Fails**
   - Unexpected state change
   - Possible external interaction
   - Pause and alert user

### Logging

All major operations logged:
- Game events
- State changes
- Actions taken
- Verification results
- Errors and warnings

## Performance Considerations

### Optimization Techniques

1. **Frame Skipping**
   - Skip frames when not needed
   - Process every Nth frame
   - Configurable frequency

2. **Region of Interest (ROI)**
   - Process only game area
   - Ignore UI outside game
   - Reduce computation

3. **Caching**
   - Cache game board position
   - Cache legal moves
   - Cache AI evaluation

4. **Asynchronous Processing**
   - Non-blocking frame capture
   - Async state updates
   - Parallel action verification

### Target Performance

- Frame capture: <20ms
- State parsing: <100ms
- AI decision: <200ms
- Action execution: <50ms
- Per-loop: ~300-400ms (feasible 2-3 FPS AI decision)

## Testing Strategy

### Unit Tests

- Game state parsing
- Move generation
- Confidence calculation
- Board detection
- Action verification

### Integration Tests

- Full mode switching
- Database operations
- API endpoints
- Voice system
- End-to-end game flow

### Test Fixtures

- Mock game screenshots
- Synthetic game states
- Pre-recorded sessions
- Test profiles

## Extension Points

### Adding a New Game Adapter

1. Create `YourGameAdapter extends BaseGameAdapter`
2. Implement 8 abstract methods
3. Define game-specific types
4. Register in adapter registry
5. Add to game selection UI

### Adding a New Feature

1. Create feature module
2. Follow existing patterns
3. Add tests
4. Update documentation
5. Integrate into dashboard

### Adding a New AI Algorithm

1. Create strategy class
2. Implement decision interface
3. Add to AI decision engine
4. Register in adapter

## Security Considerations

### Data Protection

- No sensitive game data stored
- Screenshots optional/configurable
- Sessions cleared on logout
- Profile data local

### Safe Operations

- Actions confined to game region
- Never access external files
- Respects window boundaries
- Asks before risky actions

### Limitations

- Cannot bypass CAPTCHA
- Cannot bypass anti-cheat
- Cannot authenticate
- Respects platform restrictions

## Future Architecture Improvements

1. **Machine Learning Integration**
   - Custom neural networks for game detection
   - Reinforcement learning for strategy
   - Transfer learning between games

2. **Distributed Processing**
   - Cloud-based AI decision
   - Parallel game instances
   - Shared learning

3. **Advanced Vision**
   - Real-time object detection (YOLO)
   - Pose estimation
   - Scene understanding

4. **Natural Language**
   - Voice commands
   - Conversational guidance
   - Natural input

5. **Multi-Game Support**
   - Simultaneous games
   - Game switching
   - Cross-game learning

---

**Last Updated**: October 2026
**Version**: 0.1.0
