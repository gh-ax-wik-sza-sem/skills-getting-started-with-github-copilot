# Mergington High School Activities - AI Agent Instructions

## Architecture Overview

This is a **FastAPI + vanilla JavaScript** web application for managing high school extracurricular activities.

**Tech Stack:**
- Backend: FastAPI with in-memory storage (no database)
- Frontend: Static HTML/CSS/JS served via `StaticFiles`
- Testing: pytest with `TestClient`

**Critical Design Decision:** The `activities` dict in `src/app.py` is the ONLY data store. Data resets on restart - this is intentional for a learning project. Do not suggest databases unless explicitly requested.

## Project Structure

```
src/
  app.py              # FastAPI backend - all endpoints here
  static/
    index.html        # Single-page UI
    app.js            # Frontend with async API calls
    styles.css        # Styling (no bullet points, trash emoji delete icons)
tests/
  test_app.py         # Comprehensive API tests with fixtures
.github/
  instructions/       # File-scoped instructions for Python and tests
```

## Development Workflow

**Run the app:**
```bash
python src/app.py
# Access at http://localhost:8000 (redirects to /static/index.html)
# API docs at http://localhost:8000/docs
```

**Run tests:**
```bash
pytest tests/ -v
# Uses pytest.ini with pythonpath = .
# All tests use autouse fixture to reset activities state
```

**Dependencies:** `pip install -r requirements.txt` (fastapi, uvicorn, pytest, httpx)

## Code Patterns

### Backend (FastAPI)

**API Design:**
- Activity names in path: `/activities/{activity_name}/signup`
- Email as query param: `?email=student@example.com`
- Returns JSON with `{"message": "..."}` on success
- Raises `HTTPException` for errors (404 for not found, 400 for validation)

**Endpoints:**
- `GET /activities` - Returns full activities dict
- `POST /activities/{activity_name}/signup` - Add participant
- `DELETE /activities/{activity_name}/unregister` - Remove participant
- `GET /` - Redirects to `/static/index.html`

**No async handlers:** All route functions are synchronous (in-memory ops don't need async)

### Frontend (JavaScript)

**State Refresh Pattern:** ALWAYS call `fetchActivities()` after mutations (signup/unregister) to sync UI with backend state.

**Event Delegation:** Delete buttons use delegated events on `activitiesList`:
```javascript
activitiesList.addEventListener("click", async (event) => {
  if (event.target.classList.contains("delete-btn")) {
    const activity = event.target.dataset.activity;
    const email = event.target.dataset.email;
    // ... handle deletion
  }
});
```

**UI Feedback:** Show success/error messages in `messageDiv` with `.success` or `.error` classes, auto-hide after 5 seconds.

### Testing (pytest)

**Fixture Strategy:**
- `reset_activities()` with `autouse=True` clears and repopulates activities dict before each test
- `client()` fixture returns `TestClient(app)` for API calls

**Test Organization:**
- Group by endpoint using classes: `TestSignupForActivity`, `TestUnregisterFromActivity`
- Always test URL-encoded activity names (spaces → `%20`)
- Follow Arrange-Act-Assert pattern

**Example:**
```python
def test_signup_adds_participant(self, client):
    # Arrange: (handled by reset_activities fixture)
    # Act
    client.post("/activities/Chess Club/signup?email=new@example.com")
    # Assert
    response = client.get("/activities")
    assert "new@example.com" in response.json()["Chess Club"]["participants"]
```

## Common Tasks

### Adding a New Endpoint
1. Add route handler in `src/app.py` with appropriate decorator
2. Update `src/static/app.js` to call the new endpoint
3. Create test class in `tests/test_app.py` with success/error cases
4. Update `reset_activities` fixture if new data structure needed

### Modifying Activity Schema
1. Update `activities` dict in `src/app.py`
2. Update `reset_activities()` fixture in `tests/test_app.py`
3. Update frontend rendering in `fetchActivities()` function
4. Update test assertions

## Key Conventions

- **Python:** Use snake_case, type hints, docstrings (see `.github/instructions/python.instructions.md`)
- **Tests:** pytest, descriptive names, mock external deps (see `.github/instructions/test.instructions.md`)
- **Frontend:** No bullets on participant lists (`list-style-type: none`), trash emoji (🗑️) for delete
- **Error handling:** HTTPException with appropriate status codes and descriptive messages
- **URL encoding:** Always handle spaces and special chars in activity names
