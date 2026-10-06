# Testing Guide - Universal AI Game Player

## Test Strategy

The application uses a multi-layered testing approach:

1. **Unit Tests** - Individual function/class testing
2. **Integration Tests** - Component interaction testing
3. **Manual Testing** - Real-world game testing
4. **End-to-End Tests** - Complete workflow testing

## Running Tests

```bash
# Run all tests
npm test

# Run tests in watch mode
npm test -- --watch

# Run tests with coverage
npm test -- --coverage

# Run specific test file
npm test -- game-adapters/tic-tac-toe.test.ts
```

## Unit Tests

### Game Adapter Tests

#### Tic-Tac-Toe Minimax Algorithm

```typescript
describe("TicTacToeAdapter", () => {
  const adapter = new TicTacToeAdapter();

  describe("minimax", () => {
    test("should return center cell as best first move", () => {
      // Empty board - center (index 4) is optimal
      const board = Array(9).fill(null);
      const bestMove = adapter.chooseAction(mockState(board));
      expect(bestMove.cellIndex).toBe(4);
    });

    test("should block opponent win", () => {
      // X in positions 0, 1 - should play 2 to block
      const board = ["X", "X", null, null, null, null, null, null, null];
      const bestMove = adapter.chooseAction(mockState(board));
      expect(bestMove.cellIndex).toBe(2);
    });

    test("should win when possible", () => {
      // X can win at position 2
      const board = ["X", "X", null, null, null, null, null, null, null];
      const bestMove = adapter.chooseAction(mockState(board));
      expect(bestMove.cellIndex).toBe(2);
    });
  });

  describe("detectGame", () => {
    test("should detect tic-tac-toe board", () => {
      const mockFrame = createMockTicTacToeFrame();
      expect(adapter.detectGame(mockFrame)).toBe(true);
    });

    test("should not detect non-tic-tac-toe", () => {
      const mockFrame = createMockChessFrame();
      expect(adapter.detectGame(mockFrame)).toBe(false);
    });
  });

  describe("verifyAction", () => {
    test("should verify successful action", () => {
      const prevState = mockState([...Array(9).fill(null)]);
      const currState = mockState(["X", null, null, null, null, null, null, null, null]);
      const action = { cellIndex: 0 };
      expect(adapter.verifyAction(prevState, currState, action)).toBe(true);
    });

    test("should reject invalid action", () => {
      const prevState = mockState(["X", "O", null, null, null, null, null, null, null]);
      const currState = mockState(["X", "O", null, null, null, null, null, null, null]); // No change
      const action = { cellIndex: 0 };
      expect(adapter.verifyAction(prevState, currState, action)).toBe(false);
    });
  });
});
```

### AI Coach Tests

```typescript
describe("AICoach", () => {
  const mockAdapter = new TicTacToeAdapter();
  const coach = new AICoach(mockAdapter);

  describe("analyzeAndRecommend", () => {
    test("should return recommendation with confidence", () => {
      const state = mockGameState();
      const rec = coach.analyzeAndRecommend(state);

      expect(rec).toBeDefined();
      expect(rec.action).toBeDefined();
      expect(rec.confidence).toBeGreaterThanOrEqual(0);
      expect(rec.confidence).toBeLessThanOrEqual(1);
      expect(rec.explanation).toBeDefined();
      expect(rec.reasoning).toBeDefined();
    });

    test("should throw when no legal actions available", () => {
      const fullBoard = {
        ...mockGameState(),
        availableActions: [],
      };
      expect(() => coach.analyzeAndRecommend(fullBoard)).toThrow();
    });
  });

  describe("getVoiceInstruction", () => {
    test("should format English instruction correctly", () => {
      coach.setLanguage("en");
      const rec = coach.analyzeAndRecommend(mockGameState());
      const instruction = coach.getVoiceInstruction(rec);

      expect(instruction).toContain("Confidence");
      expect(instruction).toContain("%");
    });

    test("should format Hindi instruction correctly", () => {
      coach.setLanguage("hi");
      const rec = coach.analyzeAndRecommend(mockGameState());
      const instruction = coach.getVoiceInstruction(rec);

      expect(instruction).toBeDefined();
      expect(instruction.length).toBeGreaterThan(0);
    });
  });

  describe("language switching", () => {
    test("should switch between languages", () => {
      coach.setLanguage("en");
      expect(coach.getLanguage()).toBe("en");

      coach.setLanguage("hi");
      expect(coach.getLanguage()).toBe("hi");
    });
  });

  describe("TTS toggle", () => {
    test("should enable/disable TTS", () => {
      coach.setTTSEnabled(true);
      expect(coach.isTTSEnabled()).toBe(true);

      coach.setTTSEnabled(false);
      expect(coach.isTTSEnabled()).toBe(false);
    });
  });
});
```

### Game State Manager Tests

```typescript
describe("GameStateManager", () => {
  let manager: GameStateManager;

  beforeEach(() => {
    manager = new GameStateManager();
  });

  describe("session lifecycle", () => {
    test("should start and end session", () => {
      const session = manager.startSession("tic-tac-toe", "coach");

      expect(session).toBeDefined();
      expect(session.gameType).toBe("tic-tac-toe");
      expect(session.mode).toBe("coach");
      expect(session.startTime).toBeDefined();

      const completed = manager.endSession("win");
      expect(completed?.result).toBe("win");
      expect(completed?.endTime).toBeDefined();
    });
  });

  describe("state management", () => {
    test("should track state changes", () => {
      const state1 = mockGameState();
      const state2 = mockGameState();

      manager.setState(state1);
      expect(manager.getState()).toEqual(state1);

      manager.setState(state2);
      expect(manager.getPreviousState()).toEqual(state1);
      expect(manager.getState()).toEqual(state2);
    });

    test("should notify listeners on state change", () => {
      const listener = jest.fn();
      manager.onStateChange(listener);

      const state = mockGameState();
      manager.setState(state);

      expect(listener).toHaveBeenCalledWith(state);
    });
  });

  describe("statistics", () => {
    test("should calculate session statistics", () => {
      manager.startSession("tic-tac-toe", "auto_play");

      // Add some actions
      manager.recordAction({ ...mockAction(), confidence: 0.9 });
      manager.markActionSuccessful();
      manager.recordAction({ ...mockAction(), confidence: 0.8 });
      manager.markActionFailed();

      const stats = manager.getSessionStats();

      expect(stats?.totalActions).toBe(2);
      expect(stats?.successRate).toBe(0.5); // 1 success, 1 failure
      expect(stats?.averageConfidence).toBeCloseTo(0.85, 2);
    });
  });
});
```

## Integration Tests

### API Tests

```typescript
describe("Game Profiles API", () => {
  describe("CRUD Operations", () => {
    test("should create and retrieve profile", async () => {
      const profile = {
        name: "Test Profile",
        gameName: "tic-tac-toe",
        platform: "Browser Game",
        adapter: "tic-tac-toe",
        rules: {},
        controls: {},
        objects: {},
        actions: {},
        confidenceThreshold: "0.80",
        isBuiltIn: false,
        isLearned: false,
      };

      const created = await fetch("/api/game-profiles", {
        method: "POST",
        body: JSON.stringify(profile),
      }).then((r) => r.json());

      expect(created.id).toBeDefined();
      expect(created.name).toBe(profile.name);

      const retrieved = await fetch(`/api/game-profiles/${created.id}`)
        .then((r) => r.json());

      expect(retrieved.id).toBe(created.id);
    });

    test("should update profile", async () => {
      const profile = await createTestProfile();
      const updated = await fetch(`/api/game-profiles/${profile.id}`, {
        method: "PUT",
        body: JSON.stringify({
          name: "Updated Name",
        }),
      }).then((r) => r.json());

      expect(updated.name).toBe("Updated Name");
    });

    test("should delete profile", async () => {
      const profile = await createTestProfile();
      const response = await fetch(`/api/game-profiles/${profile.id}`, {
        method: "DELETE",
      });

      expect(response.ok).toBe(true);

      const notFound = await fetch(`/api/game-profiles/${profile.id}`);
      expect(notFound.status).toBe(404);
    });
  });
});

describe("Game Sessions API", () => {
  test("should create and track session", async () => {
    const profile = await createTestProfile();
    const session = await fetch("/api/game-sessions", {
      method: "POST",
      body: JSON.stringify({
        profileId: profile.id,
        mode: "coach",
      }),
    }).then((r) => r.json());

    expect(session.profileId).toBe(profile.id);
    expect(session.mode).toBe("coach");
  });
});
```

### Component Tests

```typescript
describe("Dashboard Component", () => {
  test("should render with game selection", () => {
    const { getByText } = render(<Dashboard />);

    expect(getByText("Tic-Tac-Toe")).toBeInTheDocument();
    expect(getByText("Chess")).toBeInTheDocument();
  });

  test("should switch between coach and auto play modes", async () => {
    const { getByText } = render(<Dashboard />);

    const coachButton = getByText("AI Coach");
    fireEvent.click(coachButton);

    expect(getByText("AI Coach Mode")).toBeInTheDocument();
  });
});

describe("Coach Panel Component", () => {
  test("should display recommendations", () => {
    const mockAdapter = new TicTacToeAdapter();
    const mockCoach = new AICoach(mockAdapter);

    const { getByText } = render(
      <CoachPanel
        aiCoach={mockCoach}
        selectedGame="tic-tac-toe"
        onStop={jest.fn()}
      />
    );

    expect(getByText("AI Coach Mode")).toBeInTheDocument();
    expect(getByText(/Confidence/)).toBeInTheDocument();
  });
});
```

## Manual Testing Checklist

### Basic Functionality

- [ ] Application loads without errors
- [ ] Dashboard displays correctly
- [ ] All buttons are clickable
- [ ] Layout is responsive

### Game Connection

- [ ] Can connect to screen/region
- [ ] Live preview updates
- [ ] Can disconnect/reconnect

### Coach Mode

- [ ] Coach mode starts without errors
- [ ] Recommendations appear on screen
- [ ] Confidence scores displayed
- [ ] Voice instructions work (if enabled)
- [ ] "Speak Again" button works
- [ ] Can switch to Auto Play
- [ ] Can stop coaching

### Auto Play Mode

- [ ] Auto Play mode starts
- [ ] AI executes actions
- [ ] Action log updates
- [ ] Success/failure indicators show
- [ ] Pauses on low confidence
- [ ] Can resume after pause
- [ ] Can stop auto play

### Tic-Tac-Toe Specific

- [ ] Board is detected correctly
- [ ] X/O positions recognized
- [ ] AI plays optimally
- [ ] Game end detected
- [ ] Winner announced
- [ ] Coach explains moves clearly
- [ ] Auto Play makes valid moves

### Voice Features

- [ ] English voice output works
- [ ] Hindi voice output works
- [ ] Can toggle voice on/off
- [ ] Volume is appropriate
- [ ] Speech is clear and understandable

### Data Persistence

- [ ] Profiles saved to database
- [ ] Sessions recorded
- [ ] Statistics tracked
- [ ] Can load previous sessions
- [ ] Can export profiles

### Error Handling

- [ ] Gracefully handles game disconnect
- [ ] Shows error messages clearly
- [ ] Can recover from errors
- [ ] Emergency stop works

## Performance Testing

### Benchmarks

```typescript
describe("Performance", () => {
  test("frame capture should be < 50ms", async () => {
    const capture = new ScreenCapture();
    await capture.initialize();

    const start = performance.now();
    capture.captureFrame();
    const duration = performance.now() - start;

    expect(duration).toBeLessThan(50);
  });

  test("AI decision should be < 200ms", () => {
    const adapter = new TicTacToeAdapter();
    const state = mockGameState();

    const start = performance.now();
    adapter.chooseAction(state);
    const duration = performance.now() - start;

    expect(duration).toBeLessThan(200);
  });

  test("full decision loop should be < 400ms", async () => {
    const adapter = new TicTacToeAdapter();
    const state = mockGameState();
    const coach = new AICoach(adapter);

    const start = performance.now();
    coach.analyzeAndRecommend(state);
    const duration = performance.now() - start;

    expect(duration).toBeLessThan(400);
  });
});
```

## Test Data Fixtures

### Mock Game States

```typescript
export function mockGameState(): GameState {
  return {
    gameType: "tic-tac-toe",
    objects: [],
    players: [
      { id: 1, name: "Player 1" },
      { id: 2, name: "Player 2" },
    ],
    currentPlayer: 0,
    availableActions: [
      {
        type: "click",
        startPosition: { x: 100, y: 100 },
        confidence: 0.95,
        reasoning: "Cell 0 is empty",
      },
    ],
    score: {},
    timer: null,
    status: "playing",
    confidence: 0.85,
  };
}

export function mockTicTacToeState(board: string[]): TicTacToeState {
  return {
    ...mockGameState(),
    board: board as ("X" | "O" | null)[],
    currentPlayerSymbol: "X",
    winner: null,
  };
}
```

### Mock Frames

```typescript
export function createMockTicTacToeFrame(): ImageData {
  const canvas = document.createElement("canvas");
  canvas.width = 300;
  canvas.height = 300;
  const ctx = canvas.getContext("2d")!;

  // Draw tic-tac-toe board
  ctx.strokeStyle = "black";
  ctx.lineWidth = 2;

  // Grid lines
  for (let i = 1; i < 3; i++) {
    ctx.beginPath();
    ctx.moveTo(i * 100, 0);
    ctx.lineTo(i * 100, 300);
    ctx.stroke();

    ctx.beginPath();
    ctx.moveTo(0, i * 100);
    ctx.lineTo(300, i * 100);
    ctx.stroke();
  }

  return ctx.getImageData(0, 0, 300, 300);
}
```

## Continuous Integration

### GitHub Actions Workflow

```yaml
name: Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 18
      - run: npm ci
      - run: npm run test
      - run: npm run test:coverage
      - uses: codecov/codecov-action@v3
```

## Coverage Goals

- **Statements**: 80%+
- **Branches**: 75%+
- **Functions**: 85%+
- **Lines**: 80%+

---

**Testing Philosophy**: Write tests that verify behavior, not implementation. Focus on integration tests more than unit tests as the system is highly interconnected.

**Last Updated**: October 2026
