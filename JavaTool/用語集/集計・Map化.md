
```java
// ステータス別にTodoを分類
Map<Status, List<Todo>> grouped =
    todos.stream().collect(Collectors.groupingBy(Todo::getStatus));
```

```java
Map<Status, Long> countByStatus =
    todos.stream().collect(
        Collectors.groupingBy(
            Todo::getStatus,
            Collectors.counting()
        )
    );
```

```java
Todo found = todos.stream()
    .filter(t -> t.getId().equals(id))
    .findFirst()
    .orElseThrow(() -> new IllegalArgumentException("not found: " + id));
```

```java
boolean existsOverdue = todos.stream()
    .anyMatch(t -> t.getDueDate() != null 
    && t.getDueDate().isBefore(LocalDate.now()));

```

**APIレスポンス整形で超頻出**

[collect javadoc](https://docs.oracle.com/javase/jp/21/docs/api/java.base/java/util/stream/Stream.html#collect(java.util.stream.Collector))
[Map javadoc](https://docs.oracle.com/javase/jp/21/docs/api/java.base/java/util/Map.html)
[Collectors javadoc](https://docs.oracle.com/javase/jp/21/docs/api/java.base/java/util/stream/Collectors.html)
[groupingBy javadoc](https://docs.oracle.com/javase/jp/21/docs/api/java.base/java/util/stream/Collectors.html#groupingBy(java.util.function.Function))

