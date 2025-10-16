Perfect 👍 — since you’ve confirmed that:

```java
Map<Integer, SomeDTO> someData;
```

That means:

* the **key type** is `Integer`
* the **value type** is `SomeDTO`

---

### ✅ Correct way to refactor your loop

Original (inline loop):

```java
List keys = new ArrayList(someData.keySet());
for (Object key : keys) {
    ...
}
```

Refactored version:

```java
List<Integer> keys = new ArrayList<>(someData.keySet());
iterateKeys(keys);
```

And your method definition should be:

```java
private void iterateKeys(List<Integer> keys) {
    for (Integer key : keys) {
        SomeDTO value = someData.get(key);
        // do something with key and value
    }
}
```

---

### 💡 Notes

* `Integer` is the correct type because that’s what your map keys are.
* Always use **generics** (`List<Integer>`) to keep your code type-safe and clear.
* If `someData` is a field of your class, you can directly access it inside `iterateKeys` as shown above.
  If it’s passed in, you’d modify the method like this:

  ```java
  private void iterateKeys(Map<Integer, SomeDTO> someData, List<Integer> keys) {
      for (Integer key : keys) {
          SomeDTO value = someData.get(key);
          ...
      }
  }
  ```

Would you like me to show you how to make `iterateKeys` completely standalone (so it doesn’t rely on a class-level `someData` variable)?
