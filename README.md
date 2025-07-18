# Health-Lab


Health-Lab aims to provide a unified platform for tracking personal care routines across multiple health domains. The initial focus (Phase One) is on the scalp and facial skin domains while laying the groundwork for future expansion. A full technical outline is available in [Phase1Plan.md](Phase1Plan.md).

## Prerequisites
- **Python**: Install Python 3.10+ and a package manager such as `poetry` or `pip`.
- **Node.js & Expo**: Install Node.js (16+) and the Expo CLI for the React Native mobile app.
- **Environment Variables**: Create a `.env` file with required API keys and database settings as described in the Phase One plan.

## Environment Setup
```bash
# Clone the repository
git clone <repo-url>
cd Health-Lab

# Python setup
poetry install        # or use pip to install dependencies once defined

# Node/Expo setup
npm install -g expo-cli  # if Expo not installed
cd apps/mobile && npm install  # once the mobile app exists
```

## Running the API
When the FastAPI server is implemented you will be able to start it with:
```bash
uvicorn apps.api.main:app --reload
```
Alternatively, a Docker setup will be provided to run the API and database.

## Running the Mobile App
After installing dependencies in `apps/mobile`:
```bash
expo start
```
This will launch the React Native app in the iOS or Android simulator.

## Documentation
Further documentation and domain specifications can be found in the [docs/](docs/) directory.
