# Universal AI Game Player

A production-quality, modular, extensible web-based AI game agent that can understand and interact with games through their visible user interface.

## Project Vision

The long-term vision is to create a general-purpose AI game agent that can:

1. **Teach the user how to play games** through voice instructions (AI Coach Mode)
2. **Play games automatically** with continuous observation and decision-making (Auto Play Mode)
3. **Understand external games** through their visible interfaces without requiring source code
4. **Learn unfamiliar games interactively** through demonstrations
5. **Support multiple platforms** (Browser, Windows, Android)
6. **Extend easily** with new game adapters

## Features (Phase 1-3)

### Phase 1: Core Desktop Application

- ✅ Professional Next.js/React web interface
- ✅ Dashboard with game connection
- ✅ Screen region selection
- ✅ Live game preview
- ✅ Start/Pause/Stop controls
- ✅ Emergency stop functionality
- ✅ Application logging
- ✅ Error handling
- ✅ Configuration system
- ✅ PostgreSQL data persistence

### Phase 2: AI Coach Mode

- ✅ Real-time game observation
- ✅ Game state analysis
- ✅ Best move recommendation system
- ✅ On-screen instructions
- ✅ Text-to-Speech (English)
- ✅ Text-to-Speech (Hindi)
- ✅ Configurable voice enable/disable
- ✅ Speak Again button
- ✅ AI confidence scoring
- ✅ Action reasoning and explanation

### Phase 3: Tic-Tac-Toe Complete Implementation

- ✅ Board detection
- ✅ Grid detection
- ✅ X/O position detection
- ✅ Game state parsing
- ✅ Minimax algorithm
- ✅ Optimal move selection
- ✅ On-screen coordinate calculation
- ✅ Mouse action simulation
- ✅ Action verification
- ✅ Game state validation
- ✅ Both Coach and Auto Play modes working

## Architecture

```
┌──────────────────────┐
│      GAME APP        │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│   SCREEN CAPTURE     │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ COMPUTER VISION      │
│ OCR / OBJECT         │
│ DETECTION            │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ GAME STATE           │
│ UNDERSTANDING        │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ GAME ADAPTER         │
│ OR GENERIC AGENT     │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ AI DECISION ENGINE   │
└──────────┬───────────┘
           ↓
      ┌────┴─────┐
      ↓          ↓
┌──────────┐ ┌────────────┐
│ AI COACH │ │ AUTO PLAY  │
│   VOICE  │ │   ACTION   │
└──────────┘ └─────┬──────┘
                   ↓
          ┌────────────────┐
          │ INPUT CONTROL  │
          └───────┬────────┘
                  ↓
          ┌────────────────┐
          │ ACTION VERIFY  │
          └───────┬────────┘
                  ↓
              GAME STATE
```

## Project Structure

```
src/
├── app/
│   ├── api/
│   │   ├── game-profiles/       # Game profile management
│   │   ├── game-sessions/       # Session tracking
│   │   └── health/              # Health check
│   ├── globals.css              # Global styles
│   ├── layout.tsx               # App layout
│   └── page.tsx                 # Main page
├── components/
│   ├── dashboard.tsx            # Main dashboard UI
│   ├── coach-panel.tsx          # AI Coach mode UI
│   └── auto-play-panel.tsx      # Auto Play mode UI
├── db/
│   ├── index.ts                 # Database connection
│   └── schema.ts                # Database schema
├── lib/
│   ├── game-adapters/
│   │   ├── base-adapter.ts      # Base adapter class
│   │   ├── tic-tac-toe-adapter.ts  # Tic-tac-toe implementation
│   │   └── types.ts             # Type definitions
│   ├── ai-coach.ts              # AI Coach system
│   └── screen-capture.ts        # Screen capture utilities
└── public/                      # Static assets
```

## Technology Stack

### Frontend
- **Next.js 16.2.6** - React framework with App Router
- **React 19.2.6** - UI library
- **TypeScript 5.9.3** - Type safety
- **Tailwind CSS 4.1.17** - Styling
- **Framer Motion** - Animations
- **Lucide React** - Icons

### Backend
- **Next.js API Routes** - Serverless backend
- **PostgreSQL** - Database
- **Drizzle ORM 0.45.2** - Type-safe database access

### Game AI
- **Minimax Algorithm** - Optimal game move selection
- **Confidence Scoring** - Action reliability metrics
- **Computer Vision (Future)** - Game state detection

## Installation

### Requirements
- Node.js 18+
- PostgreSQL 12+
- npm or yarn

### Setup

```bash
# 1. Clone repository
git clone <repository-url>
cd universal-game-ai

# 2. Install dependencies
npm install

# 3. Configure environment variables
cp .env.example .env
# Edit .env with your PostgreSQL connection string

# 4. Set up database
npm run db:push

# 5. Run development server
npm run dev

# 6. Open browser
# Navigate to http://localhost:3000
```

## How to Use

### Connecting a Game

1. Open the application
2. Click **"Connect Game"** button
3. Select your game screen/region using the screen selector
4. The application will begin monitoring the selected region

### AI Coach Mode

1. Select a game (e.g., Tic-Tac-Toe)
2. Click the **🎙️ AI Coach** button
3. The AI will analyze the game state and provide recommendations:
   - **Visual instruction** on-screen
   - **Voice instruction** (optional)
   - **Reasoning** behind the recommendation
   - **Confidence score** for the recommendation
4. You perform the action manually
5. AI continues analyzing and providing guidance

**Features:**
- ✅ Configurable language (English/Hindi)
- ✅ Enable/disable voice
- ✅ Speak Again button
- ✅ Real-time game monitoring

### Auto Play Mode

1. Select a game
2. Click the **🤖 Auto Play** button
3. The AI will automatically:
   - Analyze the current game state
   - Calculate the best action
   - Execute the action (mouse/keyboard input)
   - Verify the action worked
   - Continue with next state
4. Monitor the action log and confidence scores
5. AI will pause if confidence drops below threshold

**Features:**
- ✅ Automatic action execution
- ✅ Real-time action verification
- ✅ Action logging with timestamps
- ✅ Success/failure tracking
- ✅ Confidence-based pause system
- ✅ Resumable pauses

## Supported Games

### Fully Supported (Phase 3)

#### Tic-Tac-Toe
- Board detection via vision
- Perfect play using Minimax algorithm
- Both Coach and Auto Play modes
- Real-time state tracking
- Action verification
- Coach mode: Shows optimal cell to play
- Auto Play mode: Automatically plays optimal moves

### Planned Support (Future Phases)

- Chess (with Stockfish integration)
- Connect Four
- Checkers
- Ludo
- Carrom
- Generic visual game support
- External Play Store games

## Game Compatibility Levels

### Level 1 - Fully Supported
Optimized adapters with:
- Game detection
- State detection
- Legal action detection
- Optimal AI decision-making
- Action execution
- Action verification
- Game-over detection
- Coach explanations

### Level 2 - Generic Visual Support (Future)
For games without specialized adapters:
- Screen capture analysis
- Interactive region detection
- Object detection
- Text detection (OCR)
- Action discovery
- Confidence-based filtering

### Level 3 - Unknown Game Learning (Future)
When game is not recognized:
- Interactive teaching mode
- User-guided learning
- State transition recording
- Rule inference
- Profile generation and saving

## API Endpoints

### Game Profiles

```
GET    /api/game-profiles              # List all profiles
POST   /api/game-profiles              # Create new profile
GET    /api/game-profiles/[id]         # Get profile by ID
PUT    /api/game-profiles/[id]         # Update profile
DELETE /api/game-profiles/[id]         # Delete profile
```

### Game Sessions

```
GET    /api/game-sessions              # List all sessions
POST   /api/game-sessions              # Create new session
```

### Health

```
GET    /api/health                     # Application health check
```

## Database Schema

### gameProfiles
Stores game configurations:
- Game name, platform, adapter
- Learned rules and controls
- Object and action definitions
- Win conditions
- Confidence thresholds

### gameSessions
Tracks game play sessions:
- Profile ID, mode, duration
- Result (win/loss/draw/stopped)
- Action counts and statistics
- Average confidence

### aiActions
Records individual AI actions:
- Session ID, action type
- Start/end coordinates
- Confidence score
- Verification status
- Reasoning

### gameStatistics
Aggregated statistics:
- Total games, wins, losses, draws
- Average decision time
- Average confidence
- Success rates
- Detection accuracy

## Configuration

### Settings

Located in **Settings** (⚙️ icon):

- **Voice**
  - Enable/Disable Text-to-Speech
  - Language selection (English/Hindi)
  - Voice speed control (future)
  - Volume control (future)

- **AI Behavior**
  - Confidence threshold
  - Decision time limit
  - Action retry limit
  - Pause on low confidence

- **Recording**
  - Enable/disable session recording
  - Enable/disable screenshot capture
  - Privacy settings

## Logging

The application logs to console and backend:

### Log Levels
- **INFO** - General information
- **DEBUG** - Detailed debug information
- **WARN** - Warning messages
- **ERROR** - Error messages

### Categories
- Application startup/shutdown
- Game detection and state parsing
- AI decisions and reasoning
- Action execution and verification
- Voice system activity
- Error and exception handling

## Error Handling

The application gracefully handles:

- ✅ Game not detected
- ✅ Game region lost
- ✅ Board not detected
- ✅ Poor image quality
- ✅ Unknown game
- ✅ Incorrect state recognition
- ✅ Game window moved/closed
- ✅ Input system failure
- ✅ Voice system failure
- ✅ Action verification failure
- ✅ Network timeout

**Default Behavior**: Safe pause with user notification

## Safety System

### Guarantees

The application:
- ✅ Never interacts outside selected game region
- ✅ Never accesses unrelated files
- ✅ Never attempts authentication bypass
- ✅ Stops when confidence becomes too low
- ✅ Stops when unexpected screen changes occur
- ✅ Asks user when uncertain
- ✅ Supports safe emergency stop

### Limitations

- Designed for offline games and testing environments
- Does not bypass security or anti-cheat systems
- Requires explicit game window selection
- Respects platform limitations

## Performance Characteristics

### Recommended Hardware
- CPU: Dual-core or better
- RAM: 4GB minimum (8GB recommended)
- Storage: 500MB
- Network: 1Mbps for audio

### Optimization Features
- Region of Interest (ROI) processing
- Frame skipping on demand
- Efficient caching
- Event-driven updates
- Lightweight computer vision

## Testing

### Unit Tests (Planned)
- Game state parsing
- Move generation
- Board detection
- Confidence calculation

### Integration Tests (Planned)
- Coach to Auto Play mode switching
- Database operations
- API endpoints
- Voice system integration

### Manual Testing
1. Connect to Tic-Tac-Toe game
2. Test Coach mode recommendations
3. Test Auto Play action execution
4. Verify action logging
5. Test emergency stop

## Troubleshooting

### Game Not Detected
- Ensure game window is visible
- Check if game matches supported list
- Try different screen region
- Check browser console for errors

### Voice Not Working
- Enable microphone permissions
- Check browser audio settings
- Test system speaker volume
- Verify language selection

### Auto Play Not Executing Actions
- Check if game is properly connected
- Verify confidence score (should be >0.7)
- Check action logs for failures
- Try Coach mode first to verify state detection

### Database Connection Error
- Verify `DATABASE_URL` in `.env`
- Check PostgreSQL server is running
- Verify connection string format
- Check network connectivity

## Contributing

### Adding a New Game Adapter

1. Create new file in `src/lib/game-adapters/`
2. Extend `BaseGameAdapter`
3. Implement required methods:
   - `detectGame()` - Game detection logic
   - `parseState()` - State extraction
   - `getLegalActions()` - Action generation
   - `chooseAction()` - AI decision
   - `executeAction()` - Action execution
   - `verifyAction()` - Verification
   - `isGameOver()` - Game end detection
   - `explainAction()` - Human explanation

4. Register in adapter registry
5. Add to game selection list
6. Test Coach and Auto Play modes

### Code Quality

- Use TypeScript for type safety
- Follow ESLint configuration
- Write JSDoc comments
- Add unit tests
- Maintain 80%+ test coverage

## Known Limitations

### Current (Phase 1-3)
- Only Tic-Tac-Toe fully supported
- Computer vision simplified (future enhancement)
- No Android support yet
- No voice command input yet
- Single game per session
- Browser-based only

### Future Improvements
- Advanced computer vision with YOLO
- Chess with Stockfish engine
- Multiple simultaneous games
- Android support
- Voice commands
- Game profile importing/exporting
- Advanced statistics dashboard
- Multi-language support
- Custom game adapter creation UI

## Roadmap

- **Phase 4**: Chess support with engine integration
- **Phase 5**: Connect Four with Alpha-Beta pruning
- **Phase 6**: Checkers with legal move generation
- **Phase 7**: Ludo with probability-based strategy
- **Phase 8**: Carrom with physics simulation
- **Phase 9**: Generic visual game adapter
- **Phase 10**: Unknown game learning mode
- **Phase 11**: External game integration
- **Phase 12**: Advanced game profile system

## Support

For issues, questions, or suggestions:

1. Check troubleshooting section
2. Review application logs
3. Check GitHub issues
4. Submit detailed bug report with:
   - Game name
   - Operating system
   - Browser/application version
   - Reproduction steps
   - Screenshots or logs

## License

[Your License Here]

## Acknowledgments

- Next.js and React communities
- Drizzle ORM for type-safe database access
- Tailwind CSS for styling
- Framer Motion for animations
- Web Speech API for voice

---

**Built with ❤️ for game lovers and AI enthusiasts**

Last Updated: October 2026
Version: 0.1.0 (Phase 1-3)
