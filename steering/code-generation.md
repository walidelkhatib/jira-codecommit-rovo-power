---
inclusion: always
---

# Code Generation Standards

## Core Principles

- **Readable**: Self-documenting code with clear names
- **Secure**: Follow security best practices (input validation, no hardcoded secrets)
- **Testable**: Write code that's easy to test
- **Maintainable**: Keep functions small and focused

## Security Best Practices (CRITICAL)

### Input Validation
Always validate and sanitize user input:
```python
# Good: Validate input
def create_user(name: str, email: str) -> User:
    if not name or len(name) > 100:
        raise ValueError("Name must be 1-100 characters")
    
    if not re.match(r'^[\w\.-]+@[\w\.-]+\.\w+$', email):
        raise ValueError("Invalid email format")
    
    name = html.escape(name.strip())
    email = email.lower().strip()
    return User(name=name, email=email)

# Bad: No validation
def create_user(name, email):
    return User(name=name, email=email)
```

### Secrets Management
```python
# Good: Use environment variables
import os
API_KEY = os.environ.get('API_KEY')
DB_PASSWORD = os.environ.get('DB_PASSWORD')

# Bad: Hardcoded secrets
API_KEY = "sk_live_abc123xyz"  # NEVER DO THIS
```

### SQL Injection Prevention
```python
# Good: Parameterized queries
def get_user_by_email(email: str) -> Optional[User]:
    query = "SELECT * FROM users WHERE email = %s"
    cursor.execute(query, (email,))
    return cursor.fetchone()

# Bad: String concatenation
def get_user_by_email(email):
    query = f"SELECT * FROM users WHERE email = '{email}'"  # VULNERABLE
    cursor.execute(query)
```

### XSS Prevention
```python
# Good: Escape output
from html import escape
def render_user_comment(comment: str) -> str:
    return escape(comment)

# Bad: Raw output
def render_user_comment(comment):
    return comment  # VULNERABLE
```

## Testing Standards

### Test Structure (AAA Pattern)
```python
def test_calculate_discount():
    # Arrange
    user = User(id="123", tier="gold")
    amount = 100.0
    
    # Act
    discount = calculate_discount(user, amount)
    
    # Assert
    assert discount == 10.0
```

### Test Coverage Requirements
- Test happy paths
- Test edge cases (zero, negative, null, empty)
- Test error conditions
- Aim for >80% coverage

```python
class TestUserDiscount:
    def test_positive_amount(self):
        """Test with valid amount"""
        pass
    
    def test_zero_amount(self):
        """Test with zero amount"""
        pass
    
    def test_negative_amount_raises_error(self):
        """Test that negative amount raises ValueError"""
        with pytest.raises(ValueError):
            calculate_discount(user, -10.0)
```

## Code Quality

### Type Hints (Python)
Always use type hints:
```python
from typing import List, Dict, Optional

def process_users(users: List[User]) -> Dict[str, User]:
    return {user.id: user for user in users}
```

### Type Safety (TypeScript)
```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

async function fetchUser(userId: string): Promise<User> {
  const response = await fetch(`/api/users/${userId}`);
  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }
  return await response.json();
}
```

### Error Handling
```python
# Good: Specific exceptions, proper logging
def read_config(path: str) -> Dict[str, Any]:
    try:
        with open(path, 'r') as f:
            return json.load(f)
    except FileNotFoundError:
        logger.error(f"Config not found: {path}")
        raise
    except json.JSONDecodeError as e:
        logger.error(f"Invalid JSON: {e}")
        raise ConfigError(f"Invalid config: {e}")

# Bad: Bare except, swallowing errors
def read_config(path):
    try:
        return json.load(open(path))
    except:
        return {}
```

### Function Size
- Keep functions small (< 50 lines)
- One function, one purpose
- Extract complex logic into helper functions

## Documentation

### Docstrings
```python
def process_payment(user_id: str, amount: float) -> PaymentResult:
    """Process a payment for a user.
    
    Args:
        user_id: Unique identifier for the user
        amount: Payment amount in USD
        
    Returns:
        PaymentResult with transaction ID and status
        
    Raises:
        ValueError: If amount is negative or zero
        PaymentError: If payment processing fails
    """
    pass
```

### Comments
Explain WHY, not WHAT:
```python
# Good: Explains reasoning
def calculate_tax(amount: float, state: str) -> float:
    # Use cached rates to avoid API calls on every calculation
    tax_rate = get_cached_tax_rate(state)
    
    # Round to 2 decimals to match payment processor requirements
    return round(amount * tax_rate, 2)

# Bad: States the obvious
def calculate_tax(amount, state):
    # Get tax rate
    tax_rate = get_cached_tax_rate(state)
    
    # Return amount times tax rate
    return round(amount * tax_rate, 2)
```

## Performance

### Database Queries
```python
# Good: Efficient with prefetch
def get_users_with_orders():
    return User.objects.filter(
        is_active=True
    ).prefetch_related('orders')

# Bad: N+1 query problem
def get_users_with_orders():
    users = User.objects.filter(is_active=True)
    for user in users:
        orders = user.orders.all()  # Separate query per user!
```

### Caching
```python
from functools import lru_cache

@lru_cache(maxsize=128)
def get_user_permissions(user_id: str) -> Set[str]:
    """Cache expensive permission lookups"""
    return set(Permission.objects.filter(
        user_id=user_id
    ).values_list('name', flat=True))
```

## Code Review Checklist

Before submitting code:

### Security ✅
- [ ] Input validated and sanitized
- [ ] No hardcoded secrets
- [ ] SQL injection prevented (parameterized queries)
- [ ] XSS prevented (escaped output)

### Testing ✅
- [ ] Unit tests added for new code
- [ ] Edge cases tested (zero, negative, null, empty)
- [ ] Error conditions tested
- [ ] All tests passing

### Quality ✅
- [ ] Functions are small and focused
- [ ] Clear, descriptive names
- [ ] Type hints/types added
- [ ] Error handling implemented
- [ ] No code duplication

### Documentation ✅
- [ ] Docstrings added for public functions
- [ ] Complex logic has comments explaining WHY
- [ ] README updated if needed

### Performance ✅
- [ ] No obvious performance issues
- [ ] Database queries optimized (no N+1)
- [ ] Caching used for expensive operations
