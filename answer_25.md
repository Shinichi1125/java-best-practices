Good question — and you’re right to be suspicious.
No — **you cannot just remove `query`**.
If you do, the code won’t compile (`jdbcTemplate.(...)` is invalid syntax).

Let’s unpack what’s really happening 👇

---

### 🔍 What SonarQube is telling you

SonarQube’s warning:

> “Remove this use of 'query'; it is deprecated.”

means that the **specific overload** of `JdbcTemplate.query(...)` you’re calling has been **deprecated** — not that all `query(...)` calls are bad.

You wrote:

```java
return jdbcTemplate.query(SOME_SQL_STATEMENT, new Object[][id], new SomeRowMapper());
```

That looks like a typo (`new Object[][id]`), but assuming you meant:

```java
return jdbcTemplate.query(SOME_SQL_STATEMENT, new Object[] { id }, new SomeRowMapper());
```

— this is calling an **older** overload that takes an `Object[]` as parameters.
That particular version has been deprecated in newer versions of Spring.

---

### ✅ The modern (non-deprecated) replacement

You should use a variant that takes a `PreparedStatementSetter` or varargs.

#### Option 1: Use varargs

If you’re just passing simple parameters:

```java
return jdbcTemplate.query(SOME_SQL_STATEMENT, new SomeRowMapper(), id);
```

👉 This is now the preferred way.

#### Option 2: Use `query(String, PreparedStatementSetter, RowMapper)`

```java
return jdbcTemplate.query(
    SOME_SQL_STATEMENT,
    ps -> ps.setObject(1, id),
    new SomeRowMapper()
);
```

This gives you more control and is clearer for more complex cases.

---

### ✅ Example: Before vs. After

**Before (deprecated):**

```java
return jdbcTemplate.query("SELECT * FROM users WHERE id = ?", new Object[] { id }, new UserRowMapper());
```

**After (modern, preferred):**

```java
return jdbcTemplate.query("SELECT * FROM users WHERE id = ?", new UserRowMapper(), id);
```

---

### 💡 Summary

| Old (Deprecated)                                              | New (Preferred)                                                |
| ------------------------------------------------------------- | -------------------------------------------------------------- |
| `query(String sql, Object[] args, RowMapper<T> rm)`           | `query(String sql, RowMapper<T> rm, Object... args)`           |
| `query(String sql, Object[] args, ResultSetExtractor<T> rse)` | `query(String sql, ResultSetExtractor<T> rse, Object... args)` |

---

### 🚫 Don’t do this:

```java
return jdbcTemplate.(SOME_SQL_STATEMENT, new Object[] { id }, new SomeRowMapper());
```

That’s invalid Java syntax and will not compile.

---

Would you like me to show you the **fully corrected** version of that method (including generics and return type)?
