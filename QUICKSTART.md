# Quick Start Guide - Universal AI Game Player

## 5-Minute Setup

### Prerequisites
- Node.js 18+
- PostgreSQL 12+ (or use Docker)
- Modern web browser (Chrome, Firefox, Safari, Edge)

### Installation

```bash
# 1. Clone and install
git clone <repository-url>
cd universal-game-ai
npm install

# 2. Set up environment
cp .env.example .env

# 3. Run development server
npm run dev

# 4. Open browser
# Navigate to http://localhost:3000
```

### With Docker (Alternative)

```bash
# Start everything with Docker Compose
docker-compose up

# Open http://localhost:3000
```

## First Game Session (2 minutes)

### Step 1: Connect a Game
1. Open the application
2. Click **"Connect Game"** button
3. Select your screen/game region
4. Live preview will show

### Step 2: Select Game
1. Choose **"Tic-Tac-Toe"** from the list (only fully supported game in Phase 1-3)
2. Status will change to "Tic-Tac-Toe - Connected"

### Step 3: AI Coach Mode
1. Click **"🎙️ AI Coach"** button
2. AI will analyze the game
3. Recommendation appears with:
   - Recommended action
   - Reasoning
   - Confidence score
4. Click **"🔊 Speak Again"** to hear instruction
5. You perform the action manually
6. AI continues providing recommendations

### Step 4: Auto Play Mode
1. Click **"🤖 Auto Play"** button (after connecting game)
2. AI will automatically:
   - Analyze game state
   - Execute optimal moves
   - Log each action
   - Verify results
3. Watch the action log and statistics update in real-time
4. Click **"Stop"** to end session

## Configuration

### Voice Settings
In **Coach Mode**:
- Toggle "Voice Enabled" on/off
- Select language: English or Hindi
- Instructions will be spoken automatically

### AI Settings
Default configuration:
- **Confidence Threshold**: 0.80 (auto-pause if below)
- **Decision Time**: 5000ms max
- **Language**: English
- **Voice**: Enabled

Edit in `.env` file for advanced users.

## Features at a Glance

| Feature | Status | Mode | Details |
|---------|--------|------|---------|
| Game Selection | ✅ | Both | Select from available games |
| Game Connection | ✅ | Both | Connect to game screen/region |
| Live Preview | ✅ | Both | Real-time game view |
| Coach Recommendations | ✅ | Coach | AI tells you what to do |
| Voice Instructions | ✅ | Coach | Listen to recommendations (2 languages) |
| Auto Play | ✅ | Auto | AI plays automatically |
| Action Logging | ✅ | Auto | Track all actions with timestamps |
| Statistics | ✅ | Auto | Success rate, confidence, timing |
| Pause/Resume | ✅ | Both | Pause and resume at any time |
| Emergency Stop | ✅ | Both | Stop immediately |

## Supported Games (Phase 1-3)

### Tic-Tac-Toe ✅
- **Status**: Fully Supported
- **Description**: Classic 3x3 grid game
- **AI**: Perfect play using Minimax algorithm
- **Coach Mode**: Shows optimal cell to play
- **Auto Play Mode**: Automatically plays optimal moves
- **Both Modes**: Complete game detection and verification

## Testing the Application

### Coach Mode Test (1 minute)
1. Select Tic-Tac-Toe
2. Start Coach Mode
3. Observe recommendation appears
4. Note confidence score
5. Check voice instruction (if enabled)
6. Click "Done" or "Skip"
7. Stop when done

### Auto Play Test (2 minutes)
1. Select Tic-Tac-Toe
2. Start Auto Play Mode
3. Watch AI execute moves
4. Observe action log
5. Monitor success rate
6. Check statistics
7. Watch game complete

## Common Tasks

### Enable Voice
```
Settings → Coach Panel → Toggle "Voice Enabled"
Choose Language: English or Hindi
```

### Check Statistics
```
Auto Play Mode → View "Statistics" section
Shows: Games Played, Success Rate, Confidence
```

### View Game Profiles
```
API: GET /api/game-profiles
Shows all saved game profiles
```

### Create Custom Profile
```
API: POST /api/game-profiles
Fields: name, gameName, platform, adapter
```

## Troubleshooting

### Application Won't Start
```bash
# Check Node.js version
node --version  # Should be 18+

# Check npm packages
npm install

# Clear cache
rm -rf node_modules .next
npm install
npm run build
```

### Database Connection Error
```bash
# Check PostgreSQL is running
psql --version

# Check DATABASE_URL in .env
cat .env | grep DATABASE_URL

# Test connection
psql $DATABASE_URL -c "SELECT 1"
```

### Voice Not Working
- Check browser permissions for audio
- Enable "Voice Enabled" toggle
- Check system volume
- Try different browser

### Game Not Detected
- Ensure game window is visible
- Try different screen region
- Check game list for support
- Only Tic-Tac-Toe supported in Phase 1-3

## Environment Variables

Essential variables in `.env`:

```env
# Required
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/universal_game_ai

# Optional (defaults shown)
NODE_ENV=development
NEXT_PUBLIC_VOICE_ENABLED=true
NEXT_PUBLIC_VOICE_LANGUAGE=en
AI_CONFIDENCE_THRESHOLD=0.80
```

See `.env.example` for all available variables.

## Next Steps

1. **Explore the Code**
   - Read `ARCHITECTURE.md` for system design
   - Review `src/lib/game-adapters/` for game implementation
   - Check `src/components/` for UI components

2. **Add a New Game** (Advanced)
   - Create adapter extending `BaseGameAdapter`
   - Implement required methods
   - Register in game selection
   - See `ARCHITECTURE.md` for details

3. **Deploy to Production**
   - Follow `DEPLOYMENT.md` guide
   - Supports: Vercel, Heroku, AWS, Docker

4. **Watch for Phase 4+**
   - Chess support with Stockfish
   - Connect Four
   - Checkers
   - Ludo
   - Carrom

## Key Documentation

| Document | Purpose | Audience |
|----------|---------|----------|
| `README.md` | Project overview and features | Everyone |
| `ARCHITECTURE.md` | System design and components | Developers |
| `TESTING.md` | Testing strategies and examples | QA, Developers |
| `DEPLOYMENT.md` | Production deployment guide | DevOps, SRE |
| `IMPLEMENTATION_SUMMARY.md` | What was built and status | Project Managers |
| `QUICKSTART.md` | This file - quick start guide | New Users |

## Getting Help

1. **Check Documentation**
   - Start with README.md
   - Check ARCHITECTURE.md for system details
   - Review TESTING.md for test examples

2. **Review Code Comments**
   - TypeScript files have JSDoc comments
   - Check function signatures for usage

3. **Test the API**
   ```bash
   # Health check
   curl http://localhost:3000/api/health
   
   # List profiles
   curl http://localhost:3000/api/game-profiles
   ```

4. **Check Logs**
   - Browser console (F12) for frontend errors
   - Terminal for backend logs
   - Database logs for connection issues

## Performance Tips

- **Smoother AI**: Close other applications
- **Faster Decisions**: Increase `AI_MAX_DECISION_TIME` if needed
- **Better Voice**: Ensure microphone/speakers are working
- **Stable Game**: Use stable internet for external games (future)

## Keyboard Shortcuts (Planned)

| Shortcut | Action | Status |
|----------|--------|--------|
| `Space` | Start/Pause | Future |
| `ESC` | Emergency Stop | Future |
| `V` | Toggle Voice | Future |
| `L` | Show Logs | Future |

Currently, use buttons in UI.

## Examples

### Run Coach Mode for Tic-Tac-Toe
```
1. npm run dev
2. Open http://localhost:3000
3. Select "Tic-Tac-Toe"
4. Click "Connect Game"
5. Click "🎙️ AI Coach"
6. Watch recommendations appear
7. Perform action manually
8. Continue with next moves
```

### Run Auto Play for Tic-Tac-Toe
```
1. npm run dev
2. Open http://localhost:3000
3. Select "Tic-Tac-Toe"
4. Click "Connect Game"
5. Click "🤖 Auto Play"
6. Watch AI play automatically
7. Monitor action log
8. Game completes
```

### Create a Game Profile
```bash
curl -X POST http://localhost:3000/api/game-profiles \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My TTT Profile",
    "gameName": "tic-tac-toe",
    "platform": "Browser Game",
    "adapter": "tic-tac-toe",
    "rules": {},
    "controls": {},
    "objects": {},
    "actions": {},
    "confidenceThreshold": "0.80"
  }'
```

## FAQ

**Q: Is this AI playing for me or helping me?**
A: Both! Coach Mode tells you what to do. Auto Play Mode plays automatically.

**Q: Does it work with my game?**
A: Phase 1-3 only supports Tic-Tac-Toe. Phase 4+ will add more games.

**Q: Can I use voice commands?**
A: Planned for future phases. Currently, use UI buttons.

**Q: What languages are supported?**
A: Phase 1-3 supports English and Hindi voice instructions.

**Q: Can it play online games?**
A: Yes, any visible game (browser, desktop, emulator).

**Q: Is my game data private?**
A: By default, sessions are stored locally. Screenshot recording can be disabled.

**Q: Can I export my data?**
A: Planned for Phase 4. Currently use API directly.

---

**Ready to get started?** Open http://localhost:3000 and begin! 🎮🤖

For detailed information, see the main **[README.md](README.md)** file.
