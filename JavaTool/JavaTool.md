# [[ソート]]

## sortメソッド

```java
// 優先度が高い順 → 期限が早い順
todos.sort(
    Comparator.comparing(Todo::getPriority).reversed()
              .thenComparing(Todo::getDueDate)
);
```

- todosは、Listの型を処理する。

# [[フィルタリング]]

# [[集計・Map化]]
