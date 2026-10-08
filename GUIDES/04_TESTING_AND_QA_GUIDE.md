# Testing and QA Guide — Deep Dive

Companion to Part 9 of `01_SOFTWARE_ENGINEERING_FIELD_GUIDE.md`.

---

## Table of Contents

- [1. Why testing exists](#1-why-testing-exists)
- [2. The test pyramid](#2-the-test-pyramid)
- [3. What to test](#3-what-to-test)
- [4. pytest fundamentals](#4-pytest-fundamentals)
- [5. Fixtures](#5-fixtures)
- [6. Parametrisation](#6-parametrisation)
- [7. Test doubles](#7-test-doubles)
- [8. Structure and markers](#8-structure-and-markers)
- [9. Coverage](#9-coverage)
- [10. Integration and end-to-end tests](#10-integration-and-end-to-end-tests)
- [11. Property-based testing](#11-property-based-testing)
- [12. Concurrency and race conditions](#12-concurrency-and-race-conditions)
- [13. Load and performance testing](#13-load-and-performance-testing)
- [14. Manual and exploratory testing](#14-manual-and-exploratory-testing)
- [15. Test anti-patterns](#15-test-anti-patterns)
- [16. CI integration](#16-ci-integration)
- [17. Writing tests for legacy code](#17-writing-tests-for-legacy-code)
- [18. Learning resources](#18-learning-resources)

---

# 1. Why testing exists

## 1.1 The purpose is not to find bugs

Testing **proves the absence of regressions**. It establishes that the code behaves as specified for the cases you considered.

Bugs are found by:

| Method | Finds |
|---|---|
| **Tests** | Regressions — bugs you already knew about, coming back |
| **Code review** | Logic errors, design problems, unhandled cases |
| **Static analysis** | Type errors, unreachable code, injection risks |
| **Exploratory testing** | Cases nobody thought to specify |
| **Users** | Everything above, in production, publicly |

Tests are one input among several. A suite with 100% coverage and no assertions proves nothing.

## 1.2 The economics

| When a bug is found | Relative cost to fix |
|---|---|
| While writing the test | 1× |
| During local testing | 5× |
| In CI | 10× |
| In staging | 20× |
| In production | 100× |

Not because fixing it is harder — because *discovering what broke, containing the blast radius, and reverting safely* is harder.

## 1.3 Tests as executable specifications

A test is the only documentation that cannot go stale. Prose comments rot silently; a test fails loudly the moment behaviour changes.

```python
def test_overdue_cards_are_returned_first():
    """A card due two days ago must sort before one due today."""
    ...
```

That sentence is a precise, verifiable requirement that a new developer can trust.

---

# 2. The test pyramid

```
            /\
           /  \          E2E            5–10%
          /    \         Slow, brittle, few, high confidence
         /------\
        /        \       Integration   15–25%
       /          \      Real components, moderate speed
      /------------\
     /              \    Unit          65–80%
    /                \   Fast, isolated, numerous, granular
   /------------------\
```

| Level | Scope | Speed | Count | Dependencies |
|---|---|---|---|---|
| **Unit** | One function or method | ms | Hundreds | None |
| **Component** | One class with collaborators mocked | ms | Dozens | None |
| **Integration** | Several real components together | 100ms–s | Tens | Database, filesystem |
| **E2E** | The whole system through its real interface | s–min | Few | Everything running |

**Goal: the majority of tests are fast and isolated.** A suite that takes 20 minutes will not be run before every commit, and a test suite nobody runs catches nothing.

## 2.1 The ice cream cone — the anti-pattern

```
   /\            Many unit tests are a sign of
  /  \           badly-factored code where every
 /____\          piece is entangled and cannot be
                  tested alone.
```

If a unit test needs a database, a cache, and an HTTP mock to test one function, the code is too tightly coupled. **The difficulty of testing is a direct measurement of design quality.**

---

# 3. What to test

## 3.1 The classification

| Category | What to test | Example |
|---|---|---|
| **Happy path** | The normal case works | Valid email, valid password → account created |
| **Boundary** | Exactly at the limit | Password of exactly 12 characters; 0 items; exactly 1000 items |
| **Edge** | Just outside the limit | 11 chars; 11 items; 1001 items |
| **Error path** | Invalid input, missing data | Wrong password, nonexistent ID, expired token |
| **Negative** | Things that must be rejected | Negative quantity; zero price; future date |
| **Invariant** | A property that must always hold | Balance never negative; count never exceeds total |
| **Concurrency** | Simultaneous access | Two users editing the same record |
| **Idempotency** | Repeating changes nothing | Sending the same payment key twice |

## 3.2 Boundary values

Off-by-one errors come from testing the middle of a range instead of its edges.

```python
MIN_PASSWORD = 12
MAX_CARDS_PER_DECK = 1000

@pytest.mark.parametrize("password, valid", [
    ("a" * (MIN_PASSWORD - 1), False),   # just below
    ("a" * MIN_PASSWORD, True),          # exactly at
    ("a" * (MIN_PASSWORD + 1), True),    # just above
])
def test_password_length_boundaries(password, valid):
    assert validate_password(password) == [] if valid else validate_password(password)


@pytest.mark.parametrize("count, accepted", [
    (0, False),
    (1, True),
    (MAX_CARDS_PER_DECK, True),
    (MAX_CARDS_PER_DECK + 1, False),
])
def test_card_count_boundaries(count, accepted):
    assert (validate_card_count(count) == []) is accepted
```

**Remember: 0 and 1 are boundaries too.** Empty lists, empty strings, and single elements break more code than large values.

## 3.3 Invariants — the most valuable tests

An invariant must hold after **every** operation, no matter the sequence.

```python
def test_bank_balance_never_negative_under_random_operations():
    """Property test: no sequence of operations should ever overdraw."""
    account = BankAccount(balance=1000)

    operations = [
        ("withdraw", 100),
        ("withdraw", 2000),      # rejected
        ("deposit", 50),
        ("withdraw", 900),
        ("withdraw", 1),
    ]

    for operation, amount in operations:
        getattr(account, operation)(amount)
        assert account.balance >= 0, f"{operation}({amount}) over drew"


def test_deck_card_count_matches_actual_cards():
    """An invariant across the repository layer."""
    deck = create_deck(owner)
    for i in range(10):
        add_card(deck, front=f"Q{i}", back=f"A{i}")

    assert deck.card_count == 10
    assert deck.card_count == session.query(Card).filter_by(deck_id=deck.id).count()
```

---

# 4. pytest fundamentals

## 4.1 Basic structure

```python
def test_addition():
    assert 2 + 2 == 4


def test_raises_value_error_on_empty():
    with pytest.raises(ValueError, match="cannot average"):
        average([])
```

## 4.2 The assert statement

```python
def test_assertion_messages():
    result = calculate_discount(100, 0.15)
    assert result == 85, f"expected 85, got {result}"      # shown only on failure


def test_approx_for_floats():
    """Never assert exact equality on floats."""
    assert calculate_discount(100, 0.15) == pytest.approx(85.0)
    assert divide(1, 3) == pytest.approx(0.3333, abs=1e-4)


def test_approx_relative_for_large_numbers():
    assert 1e9 + 1 == pytest.approx(1e9, rel=1e-9)          # absolute tolerance is useless at this scale
```

**Floating point cannot represent most decimals exactly.** `0.1 + 0.2 != 0.3` in every language. `pytest.approx` is the correct tool.

## 4.3 Useful assertions

```python
def test_rich_assertions():
    assert [1, 2, 3] == [1, 2, 3]
    assert {"a": 1} == {"a": 1}

    with pytest.raises(ValueError):
        int("not a number")

    # pytest.approx, pytest.raises, pytest.warns, pytest.importorskip
```

## 4.4 Running and selecting tests

```bash
pytest                          # everything
pytest tests/test_api.py        # one file
pytest -k "login or register"   # by name
pytest -m "not slow"            # by marker
pytest -x                       # stop at first failure
pytest --lf                     # rerun last failures
pytest --ff                     # run failures first, then the rest
pytest -vv --tb=short           # verbose, short tracebacks
pytest --durations=10           # the 10 slowest tests
pytest -p no:randomly           # disable random ordering
```

`--durations` is the highest-value flag. Slow tests get skipped, or get fixed.

---

# 5. Fixtures

## 5.1 What they do

A fixture provides setup and teardown. pytest handles nested dependencies and cleans up in reverse order automatically.

```python
import pytest


@pytest.fixture
def sample_user():
    """Available to any test requesting it by name."""
    return User(id=1, email="test@example.com", role="viewer")


@pytest.fixture
def admin_user(sample_user):
    """Depends on another fixture. sample_user is set up first,
    torn down after."""
    sample_user.role = "admin"
    return sample_user
```

## 5.2 Scope

| Scope | Created | Use for |
|---|---|---|
| `function` (default) | Per test | Independent, cheap setup |
| `class` | Per test class | Shared within a class |
| `module` | Per test file | Expensive setup done once per file |
| `session` | Once per run | Truly global: database container, config |

```python
@pytest.fixture(scope="session")
def docker_postgres():
    """Started once for the entire test run. Very expensive, so scope it wide."""
    container = start_postgres_container()
    yield f"postgresql://test:{container.password}@localhost:{container.port}/test"
    container.stop()


@pytest.fixture
def db_session(docker_postgres):
    """Per-test transaction, rolled back automatically. Tests stay isolated."""
    engine = create_engine(docker_postgres)
    connection = engine.connect()
    transaction = connection.begin()

    session = Session(bind=connection)
    Base.metadata.create_all(session.bind)

    yield session

    session.close()
    transaction.rollback()      # ← this is what guarantees isolation
    Base.metadata.drop_all(connection)
```

**The transaction-rollback pattern is the standard way to get fast, isolated database tests.** No manual cleanup, no order dependence, no shared state.

## 5.3 yield and teardown

```python
@pytest.fixture
def temp_db():
    db = create_in_memory_db()
    yield db                      # test runs here
    db.dispose()                  # teardown always runs, even if the test fails
```

## 5.4 Parameters and factory fixtures

```python
@pytest.fixture
def db(docker_postgres):
    ...                          # session setup


@pytest.fixture
def user_factory(db):
    """A factory, not a fixed object. Each test creates exactly what it needs."""
    def _make(email="test@example.com", role="viewer", **kwargs):
        user = User(email=email, role=role, **kwargs)
        db.add(user)
        db.commit()
        return user
    return _make


def test_user_defaults(user_factory):
    user = user_factory()
    assert user.role == "viewer"


def test_admin_can_do_thing(client, user_factory):
    admin = user_factory(email="admin@test.com", role="admin")
    ...
```

**Build factory fixtures, not shared object fixtures.** They eliminate inter-test coupling, which is the main cause of "it passes alone but fails in the suite."

## 5.5 Autouse

```python
@pytest.fixture(autouse=True)
def reset_settings():
    """Runs for every test in this module without being requested."""
    original = settings.dict()
    yield
    settings.update(original)      # prevent state leaking between tests
```

---

# 6. Parametrisation

The highest value-per-line feature in pytest.

```python
@pytest.mark.parametrize("password, expected_valid", [
    ("correct-horse-battery", True),
    ("short", False),
    ("", False),
    ("x" * 128, True),
    ("x" * 129, False),
])
def test_password_validation(password, expected_valid):
    assert validate_password(password) == [] if expected_valid else bool(validate_password(password))
```

## 6.1 Multiple parameters

```python
@pytest.mark.parametrize("base, exponent, expected", [
    (2, 0, 1),
    (2, 10, 1024),
    (10, 3, 1000),
])
def test_power(base, exponent, expected):
    assert base ** exponent == expected


@pytest.mark.parametrize("cards, expected_count", [
    ([], 0),
    ([{"due": False}], 1),
    ([{"due": True}, {"due": True}], 2),
])
def test_due_card_count(cards, expected_count):
    assert count_due(cards) == expected_count
```

## 6.2 Generate many cases

```python
@pytest.mark.parametrize("n", range(0, 30))
def test_factorial_of_small_numbers(n):
    assert factorial_iterative(n) == factorial_naive(n)


@pytest.mark.parametrize("size", [10, 100, 1_000, 10_000])
def test_sorting_matches_reference(size, random):
    import random as r
    data = [r.randint(0, 10_000) for _ in range(size)]

    merge_sort(data[:])
    assert merge_sort(data[:]) == sorted(data)
```

## 6.3 Parametrising with IDs

```python
@pytest.mark.parametrize(
    "value, expected",
    [
        pytest.param(0, "zero", id="zero"),
        pytest.param(-1, "negative", id="negative"),
        pytest.param(150, "oversized", id="above-limit"),
    ],
)
def test_size_classification(value, expected):
    assert classify(value) == expected
```

Readable test IDs turn a failing test from `test_x[1-2]` into `test_x[above-limit]`. That saves real debugging time.

---

# 7. Test doubles

## 7.1 The taxonomy

| Double | Replaces | Records calls | Purpose |
|---|---|---|---|
| **Dummy** | A real object | No | Satisfy a signature |
| **Stub** | A real object | No | Return canned responses |
| **Spy** | A real object | Yes | Assert something was called |
| **Mock** | A real object | Yes | Assert exact interactions |
| **Fake** | A working simplified version | No | In-memory database, etc. |

**Prefer fakes and stubs.** Mocks that assert interactions make refactoring painful — they break when you change *how* something is called, even if the behaviour is identical. Assert on **observable results**, not on call sequences.

## 7.2 `unittest.mock` essentials

```python
from unittest.mock import MagicMock, patch, call

def test_sends_welcome_email():
    mailer = MagicMock()
    register_user("test@example.com", mailer=mailer)

    mailer.send.assert_called_once()
    assert mailer.send.call_args[0][0] == "test@example.com"


def test_email_contains_name():
    mailer = MagicMock()
    register_user("test@example.com", name="Alice", mailer=mailer)

    subject = mailer.send.call_args.kwargs["subject"]
    assert "Alice" in subject


def test_never_sends_when_disabled(monkeypatch):
    monkeypatch.setattr("app.config.MAIL_ENABLED", False)
    mailer = MagicMock()

    register_user("test@example.com", mailer=mailer)

    mailer.send.assert_not_called()
```

## 7.3 Patching at the right target

```python
# ✗ WRONG — patches the name where it is defined, which your code
#            may not be looking up. Usually raises or does nothing.
with patch("app.services.email.send"): ...

# ✓ CORRECT — patch where it is USED
with patch("app.services.register_user.send_email"): ...
```

**Rule:** patch `"module_under_test.function_name"`, where `module_under_test` is the module whose code is executing.

## 7.4 Fakes over mocks

```python
class InMemoryMailer:
    """A fake, not a mock. Tests read naturally and survive refactoring."""
    def __init__(self):
        self.sent: list[tuple[str, str]] = []

    def send(self, to: str, subject: str = "", body: str = "") -> None:
        self.sent.append((to, subject or body))


def test_registration_sends_welcome_email():
    mailer = InMemoryMailer()

    register_user("test@example.com", mailer=mailer)

    assert len(mailer.sent) == 1
    assert mailer.sent[0][0] == "test@example.com"
```

Compare that with the mock version. It is shorter, clearer, and does not break when you refactor the internal call.

## 7.5 A fake in-memory database

```python
class FakeUserRepository:
    """Implements the same interface as the real repository."""
    def __init__(self):
        self._store: dict[int, User] = {}
        self._next_id = 1

    def create(self, email: str, role: str = "viewer") -> User:
        user = User(id=self._next_id, email=email, role=role)
        self._store[user.id] = user
        self._next_id += 1
        return user

    def get(self, user_id: int) -> User | None:
        return self._store.get(user_id)

    def get_by_email(self, email: str) -> User | None:
        return next((u for u in self._store.values() if u.email == email), None)

    def update(self, user: User) -> User:
        self._store[user.id] = user
        return user

    def delete(self, user_id: int) -> None:
        self._store.pop(user_id, None)
```

**This is the payoff for the repository pattern.** Because business logic depends on an interface, not on SQLAlchemy, it can be tested against a fake with no database at all.

---

# 8. Structure and markers

## 8.1 conftest.py

`conftest.py` holds fixtures shared across a directory tree.

```
tests/
├── conftest.py                 ← available everywhere below
├── unit/
│   ├── conftest.py             ← available in unit/ only
│   ├── test_domain.py
│   └── test_services.py
└── integration/
    ├── conftest.py
    └── test_api.py
```

## 8.2 Markers

```python
# pyproject.toml
[tool.pytest.ini_options]
markers = [
    "slow: takes more than a second",
    "integration: needs external services",
    "e2e: full system test",
    "security: security regression test",
    "wip: work in progress",
]
```

```python
@pytest.mark.slow
def test_large_dataset_query():
    ...


@pytest.mark.integration
def test_real_database_round_trip(db_session):
    ...


@pytest.mark.security
def test_cannot_read_another_users_deck(client):
    ...
```

```bash
pytest -m "not slow"          # fast suite for every commit
pytest -m "integration"       # full suite before merging
pytest -m "security"          # security regressions only
```

## 8.3 Test that the test suite works

A suite that silently passes everything is worse than no suite.

```python
def test_mutating_algorithm_detects_bug(mutation_checker):
    """
    Sanity check on the test infrastructure itself.
    Break the implementation; the tests should fail.
    """
    with mutation_checker.mutate("app.domain.scheduling.next_interval", "return 999"):
        result = pytest.main(["tests/unit/test_scheduling.py", "-q"])
        assert result != 0, "the suite passed against broken code — tests are useless"
```

```python
# Smoke test: assert the thing you already believe, then break it deliberately once
def test_smoke_assertion_framework_works():
    assert True
    assert 1 + 1 == 2
    # If this file ever fails, something is deeply wrong with the environment.
```

---

# 9. Coverage

## 9.1 Measuring

```bash
pytest --cov=. --cov-report=term-missing
pytest --cov=. --cov-report=html --cov-report=xml
```

```
Name                 Stmts   Miss  Cover   Missing
------------------------------------------------
app/domain/schedule    48      2    96%   45-46
app/api/routes         112     18    84%   67, 89-103
------------------------------------------------
TOTAL                  340     31    91%
```

## 9.2 Reasonable targets

| Layer | Target | Rationale |
|---|---|---|
| Business / domain logic | **90–100%** | This is where correctness matters and testing is cheap |
| Repositories | 70–85% | Mostly plumbing; test the tricky queries |
| API routes | 70–85% | Test status codes, auth behaviour, error cases |
| UI components | 50–70% | Test behaviour, not markup |
| Config / scripts | Minimal | Not worth it |

## 9.3 Coverage is a diagnostic, not a goal

```python
# ✗ 100% coverage, zero value
def test_add():
    result = add(1, 2)
    assert result is not None        # passes even if add returns "banana"


# ✓ Lower coverage, real value
def test_add():
    assert add(1, 2) == 3
    assert add(-1, 1) == 0
    assert add(0.1, 0.2) == pytest.approx(0.3)
```

**The metric to watch is not coverage but "uncovered branches."** Find which `if` branches were never taken — those are the untested error paths where bugs hide.

```bash
pytest --cov=. --cov-branch      # measure branch coverage, not just line coverage
```

---

# 10. Integration and end-to-end tests

## 10.1 FastAPI integration tests

```python
import pytest
from fastapi.testclient import TestClient
from app.main import app
from app.repositories.user_repo import UserRepository


@pytest.fixture
def client(db_session):
    """Dependency override: the app uses the test database."""
    def override_get_db():
        yield db_session
    app.dependency_overrides[get_db] = override_get_db

    with TestClient(app) as test_client:
        yield test_client

    app.dependency_overrides.clear()      # ← without this, tests leak into each other


@pytest.fixture
def auth_client(client, user_factory):
    """A logged-in client, with the token injected automatically."""
    user = user_factory(email="test@example.com", password=hash_password("valid-password-123"))

    response = client.post("/api/v1/auth/login",
                           json={"email": "test@example.com", "password": "valid-password-123"})
    assert response.status_code == 200

    token = response.json()["access_token"]
    client.headers.update({"Authorization": f"Bearer {token}"})
    return client


def test_create_and_read_deck(auth_client):
    created = auth_client.post("/api/v1/decks", json={"title": "Biology 101"})
    assert created.status_code == 201
    deck_id = created.json()["id"]

    fetched = auth_client.get(f"/api/v1/decks/{deck_id}")
    assert fetched.status_code == 200
    assert fetched.json()["title"] == "Biology 101"


def test_validation_error_shape(auth_client):
    response = auth_client.post("/api/v1/decks", json={"title": ""})

    assert response.status_code == 422
    body = response.json()
    assert body["detail"][0]["field"] == "title"      # stable, parseable error contract


def test_requires_authentication(client):
    response = client.get("/api/v1/decks")
    assert response.status_code == 401


def test_health_endpoint(client):
    response = client.get("/api/health")
    assert response.status_code == 200
    assert response.json()["status"] == "ok"
```

## 10.2 The override cleanup

`app.dependency_overrides.clear()` is mandatory. Forgetting it means test A's database leaks into test B, producing failures that depend on execution order — the hardest class of test bug to diagnose.

## 10.3 E2E tests

Test a complete user journey through the real interface.

```python
def test_complete_user_journey(auth_client):
    """Register → create deck → add cards → study → review."""

    # Create a deck
    deck = auth_client.post("/api/v1/decks", json={"title": "World Capitals"}).json()

    # Add cards
    for question, answer in [("Capital of Japan?", "Tokyo"),
                             ("Capital of France?", "Paris"),
                             ("Capital of Kenya?", "Nairobi")]:
        response = auth_client.post(
            "/api/v1/cards",
            json={"deck_id": deck["id"], "front": question, "back": answer},
        )
        assert response.status_code == 201

    # Study the deck
    session = auth_client.post(f"/api/v1/decks/{deck['id']}/study").json()
    assert len(session["cards"]) == 3

    # Answer each card
    for card in session["cards"]:
        response = auth_client.post(
            f"/api/v1/cards/{card['id']}/review",
            json={"was_correct": True, "time_taken_ms": 2000},
        )
        assert response.status_code == 200

    # Verify the summary
    summary = auth_client.get(f"/api/v1/decks/{deck['id']}/summary").json()
    assert summary["reviewed"] == 3
    assert summary["correct"] == 3
```

## 10.4 E2E discipline

| Rule | Reason |
|---|---|
| Keep them few | Each one costs minutes and is brittle |
| No `sleep()` — wait for a condition | Timing-based waits are the flakiness source |
| Test behaviour, not implementation | Refactoring internals should not break them |
| Give them clear names | A failure should tell you what broke |
| Fix or delete immediately | A permanently failing E2E test trains the team to ignore it |
| Run them on a schedule, not every commit | Too slow for the inner loop |

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC


def wait_for_element(driver, selector: str, timeout: int = 10):
    """Wait for a condition, never sleep."""
    return WebDriverWait(driver, timeout).until(
        EC.presence_of_element_located((By.CSS_SELECTOR, selector))
    )


def test_login_page_loads(driver):
    driver.get("https://test.example.com/login")
    element = wait_for_element(driver, "#email-input")
    assert element.is_displayed()
```

---

# 11. Property-based testing

Example-based testing checks cases **you** thought of. Property-based testing generates cases automatically and checks a property that must always hold.

```python
from hypothesis import given, strategies as st


@given(st.lists(st.integers(min_value=-1000, max_value=1000), max_size=100))
def test_reverse_twice_is_identity(numbers):
    """Sorting twice must give the same result. Property, not example."""
    assert sorted(sorted(numbers)) == sorted(numbers)


@given(st.lists(st.integers(), min_size=1))
def test_merge_sort_matches_builtin(numbers):
    assert merge_sort(numbers[:]) == sorted(numbers)


@given(st.integers(min_value=1, max_value=10**6))
def test_factorial_is_positive(n):
    assert factorial(n) > 0


@given(st.text())
def test_password_hash_never_equals_plaintext(password):
    """Security invariant: the hash must never equal the input."""
    hashed = hash_password(password)
    assert hashed != password
    assert verify_password(hashed, password)


@given(st.binary(min_size=1))
def test_encrypt_decrypt_round_trip(plaintext):
    """Encrypt then decrypt must return the original — for ANY input."""
    encryptor = FieldEncryptor(key)
    assert encryptor.decrypt(encryptor.encrypt(plaintext)) == plaintext


@given(st.lists(st.integers()))
def test_two_sum_properties(numbers):
    target = 7
    result = two_sum(numbers, target)

    if result is None:
        assert not any(a + b == target for i, a in enumerate(numbers) for b in numbers[i+1:])
    else:
        i, j = result
        assert numbers[i] + numbers[j] == target
```

**Hypothesis finds bugs you would never have thought to test for**, because it tries hundreds of generated inputs including the awkward ones.

```bash
pip install hypothesis
hypothesis --hypothesis-profile ci tests/
```

## 11.1 When to use it

| Situation | Use Hypothesis |
|---|---|
| Pure functions with clear invariants | Yes, always |
| Parsers, serializers, encoders | Yes |
| Cryptographic round-trips | Yes |
| Scheduling / date logic | Yes |
| Anything where you are unsure of the edge cases | Yes |
| UI rendering | No |
| Third-party network calls | No — nondeterministic |

---

# 12. Concurrency and race conditions

These are the bugs that never reproduce in a test suite and always appear in production.

```python
def test_concurrent_withdrawals_do_not_overdraw():
    """
    Ten threads withdraw from a 1000-balance account.
    Only three should succeed (100 each). The rest must be rejected.
    """
    account = BankAccount(balance=1000)
    barrier = threading.Barrier(10)
    succeeded: list[bool] = []

    def attempt_withdrawal():
        barrier.wait()                        # maximise the chance of overlap
        try:
            account.withdraw(100)
            succeeded.append(True)
        except InsufficientFunds:
            succeeded.append(False)

    threads = [threading.Thread(target=attempt_withdrawal) for _ in range(10)]
    for t in threads:
        t.start()
    for t in threads:
        t.join()

    assert succeeded.count(True) == 3
    assert succeeded.count(False) == 7
    assert account.balance == 700          # ← the real assertion
```

## 12.1 The lost update problem

```python
# ✗ WRONG — read-modify-write without atomicity
def increment_counter(counter_id: int) -> None:
    counter = session.query(Counter).get(counter_id)
    counter.value += 1        # two concurrent requests both read 5, both write 6
    session.commit()


# ✓ RIGHT — let the database do it atomically
from sqlalchemy import text

def increment_counter(counter_id: int) -> None:
    session.execute(
        text("UPDATE counters SET value = value + 1 WHERE id = :id"),
        {"id": counter_id},
    )
    session.commit()
```

## 12.2 Optimistic concurrency control

```python
class Deck(Base):
    __tablename__ = "decks"

    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str]
    version: Mapped[int] = mapped_column(default=1)     # optimistic lock


def update_deck(deck_id: int, new_title: str, expected_version: int) -> Deck | None:
    deck = session.get(Deck, deck_id)
    if deck is None or deck.version != expected_version:
        return None                      # someone else changed it → caller retries

    deck.title = new_title
    deck.version += 1
    session.commit()
    return deck


def test_concurrent_update_is_rejected():
    deck = create_deck("Original")
    current_version = deck.version

    first = update_deck(deck.id, "First edit", current_version)
    assert first is not None

    second = update_deck(deck.id, "Second edit", current_version)   # same expected version
    assert second is None                  # correctly rejected
```

---

# 13. Load and performance testing

## 13.1 Locust

```python
from locust import HttpUser, task, between


class DeckApiUser(HttpUser):
    wait_time = between(1, 3)

    def on_start(self):
        response = self.client.post("/api/v1/auth/login",
                                    json={"email": "load@test.com", "password": "test-password"})
        self.token = response.json()["access_token"]
        self.headers = {"Authorization": f"Bearer {self.token}"}

    @task(3)
    def list_decks(self):
        self.client.get("/api/v1/decks", headers=self.headers)

    @task(1)
    def create_card(self):
        self.client.post("/api/v1/cards",
                         json={"deck_id": 1, "front": "Q", "back": "A"},
                         headers=self.headers)
```

```bash
locust -f locustfile.py --host https://test.example.com
```

## 13.2 What to measure

| Metric | Meaning | Threshold example |
|---|---|---|
| **p50 latency** | Median response time | < 50ms |
| **p95 latency** | 95th percentile — what users feel | < 300ms |
| **p99 latency** | Tail latency — the worst experience | < 1s |
| **Throughput** | Requests/second sustained | 500 rps |
| **Error rate** | Failed requests under load | < 0.1% |
| **Saturation point** | Where latency degrades sharply | Document it |

**Percentiles, not averages.** An average of 100ms can hide 10% of requests taking 5 seconds. Users notice the tail, not the mean.

## 13.3 The N+1 problem

The most common performance bug in web applications.

```python
# ✗ N+1 — one query for the list, then one per item
def list_decks(user_id: int):
    decks = session.query(Deck).filter_by(owner_id=user_id).all()
    return [{
        "id": d.id,
        "title": d.title,
        "card_count": d.cards.count(),      # ← fires a SELECT for EVERY deck
    } for d in decks]


# ✓ Eager loading — one query
def list_decks(user_id: int):
    decks = session.execute(
        select(Deck)
        .options(selectinload(Deck.cards))          # ← one extra query total
        .filter(Deck.owner_id == user_id)
    ).scalars().all()
    return [{"id": d.id, "title": d.title,
             "card_count": len(d.cards)} for d in decks]
```

```python
def test_list_decks_does_not_perform_n_plus_one(django_assert_num_queries, db_session):
    """Exactly 2 queries regardless of how many decks exist."""
    user = create_user()
    for i in range(50):
        create_deck(user, f"Deck {i}")

    with django_assert_num_queries(2):
        list_decks(user.id)
```

## 13.4 Indexing

```sql
-- Before: full table scan
EXPLAIN SELECT * FROM cards WHERE deck_id = 42 AND due_at <= NOW();

-- After: index lookup
CREATE INDEX idx_cards_deck_due ON cards (deck_id, due_at);

EXPLAIN SELECT * FROM cards WHERE deck_id = 42 AND due_at <= NOW();
```

**Composite index column order matters.** For `WHERE deck_id = ? AND due_at <= ?`, put the equality column first and the range column last — a range predicate before an equality cannot use the index efficiently.

---

# 14. Manual and exploratory testing

Automation cannot judge some things. Every release still needs a human.

## 14.1 Exploratory testing

**Charter:** a one-sentence mission with a time box.

> "Explore the deck editor for 45 minutes. Focus on error handling and what happens with unusual input."

Then: design a test, run it, observe, **design the next test based on what you observed.** Not a scripted sequence — genuine exploration.

## 14.2 Questions a human must answer

- [ ] Does the error message tell the user what to do next?
- [ ] Is anything confusing without explanation?
- [ ] Is the primary action obvious?
- [ ] Does it work on a small phone screen?
- [ ] Does it work with a keyboard only? (accessibility)
- [ ] What happens on a slow connection?
- [ ] What happens if the user presses back mid-submit?
- [ ] Is the destructive action clearly marked?
- [ ] Does the copy use the user's vocabulary or internal jargon?

## 14.3 The pre-release checklist

```
[ ] Happy path works end to end
[ ] Empty states are informative, not blank
[ ] Loading states exist (no infinite spinner)
[ ] Error states are actionable
[ ] Validation messages are specific
[ ] Long content does not break layout
[ ] Keyboard navigation works
[ ] Mobile viewport works
[ ] Slow network degrades gracefully
[ ] Destructive actions confirm
[ ] Refresh does not lose work
[ ] Session expiry is handled
[ ] Logs contain no secrets or PII
```

---

# 15. Test anti-patterns

| Anti-pattern | Why it is harmful | Instead |
|---|---|---|
| **No assertions** | Passes regardless of behaviour | Assert on the observable result |
| **Asserting on mocks only** | Breaks on any refactor | Assert on outcomes; fake the boundary |
| **Order-dependent tests** | Fail unpredictably | Isolated fixtures, transaction rollback |
| **Shared mutable fixtures** | Couples every test | Factory fixtures |
| **Testing implementation details** | Blocks refactoring | Test public behaviour |
| **`sleep()` in tests** | Flaky under load | Wait for a condition |
| **One giant test** | Unclear which assertion failed | One behaviour per test |
| **No error-path tests** | Crashes in production | Test every failure mode |
| **Chasing 100% coverage** | Writable, meaningless tests | Cover boundaries and branches |
| **Copy-pasted tests** | Divergence over time | Parametrise |
| **Skipping the mock cleanup** | Leaks between tests | Always clear overrides |
| **Skipping the database cleanup** | Order dependence | Transaction rollback |

## 15.1 The mirror test

> **If your test suite would still pass if you deleted the entire implementation, your tests are worthless.**

Break the implementation deliberately and confirm the suite goes red. If it stays green, your tests are measuring nothing.

---

# 16. CI integration

```yaml
name: Tests

on: [push, pull_request]

jobs:
  unit:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.11", "3.12"]

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
          cache: pip

      - run: |
          pip install -e ".[dev]"
          pytest tests/unit -m "not slow" --cov=. --cov-report=xml -q

      - name: Check coverage threshold
        run: |
          pip install coverage
          coverage report --fail-under=80

      - uses: actions/upload-artifact@v4
        if: always()          # upload even on failure — you want to debug
        with:
          name: coverage
          path: coverage.xml

  integration:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: test
          POSTGRES_USER: test
          POSTGRES_DB: testdb
        ports: ["5432:5432"]
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.12", cache: pip }
      - run: pip install -e ".[dev]"
      - run: pytest tests/integration -q
```

## 16.1 Pipeline ordering

```
lint ──▶ type check ──▶ unit tests ──▶ integration ──▶ security scan ──▶ build
```

Fail fast. Linting takes 2 seconds; running a 10-minute test suite before a lint error wastes everyone's time.

## 16.2 Flaky test management

| Response | Correctness |
|---|---|
| Ignore it | The suite becomes untrustworthy; nobody reads failures |
| Retry it | Hides the real cause |
| **Quarantine and fix** | **Correct** — remove from CI, add an owner and a deadline |

```python
@pytest.mark.flaky(reruns=2)
def test_eventually_consistent_thing():
    """Temporary. Add a comment with the issue link and a fix deadline."""
    ...
```

---

# 17. Writing tests for legacy code

When inheriting code with no tests, or if you are changing untested code:

## 17.1 The safe-change workflow

```
1. CHARACTERISE   Write tests that document what the code DOES now
                   — including the bugs. Do not "fix" anything yet.
2. REFACTOR       Change the structure. Tests must still pass.
3. FIX            Correct the bugs. Update the tests deliberately.
```

**Never combine steps 2 and 3.** If you refactor and fix simultaneously, a failing test is ambiguous — was it your refactor or your fix?

## 17.2 Characterisation tests

```python
def test_legacy_discount_behaviour():
    """
    Characterisation test. This documents EXISTING behaviour,
    including what appears to be a bug (see the second assert).

    Do not "fix" this without a deliberate decision.
    """
    assert calculate_discount(100, 0.10) == 90
    assert calculate_discount(100, 0.10) == 80     # ← looks wrong, but it IS the current behaviour
```

## 17.3 Coverage-first strategy

When you do not know where the bugs are, find them with coverage:

1. Run the existing test suite with coverage
2. Look for low-coverage modules that are complex and heavily used
3. **Those are your risk areas** — write tests there first
4. Then work outward from what you have covered

Priority: high complexity × high usage × low coverage.

---

# 18. Learning resources

| Resource | Notes |
|---|---|
| pytest official documentation | Excellent, especially the fixture and parametrisation sections |
| `pytest` real-world examples | Practical patterns |
| *Test-Driven Development* (Beck) | The original — the red/green/refactor discipline |
| *xUnit Test Patterns* (Meszaros) | The catalogue of test smells and their fixes |
| Hypothesis documentation | The best introduction to property-based testing |
| FastAPI testing guide | Official, with `TestClient` and dependency override patterns |
| mutation testing tools (`mutmut`, `cosmic-ray`) | Verify your tests actually detect bugs |
| `coverage.py` | The standard coverage tool |

## The habit that matters

> **Write the test that reproduces the bug before you fix the bug.**

Not after. Before. Because writing it first proves the bug exists, proves your fix works, and proves the bug can never return. Skipping this step leaves the same bug one refactor away from returning — and you will not remember it.

---

## Closing

Tests are not a phase you do at the end, a metric to optimise, or a safety net that removes the need to think. They are the executable form of your specification and the only thing standing between a refactor and a silent regression.

Write the failing test. Make it pass. Keep it. Then write the one that would catch the bug you have not thought of yet.
