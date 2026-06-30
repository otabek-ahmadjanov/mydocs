# Strong, Weak, Soft, and Phantom References

- **Strong Reference**

  Это тип ссылки по умолчанию. Любой объект, имеющий активную сильную ссылку, не подлежит сборке мусора.

    ```java
    MyClass obj = new MyClass ();
    ```

  Объект удаляется сборщиком мусора только тогда, **когда переменная, на которую была сделана сильная ссылка, указывает на null.**

    ```java
    obj = null; 
    //'obj' больше не ссылается на экземпляр. Таким образом, 'MyClass' теперь доступен для сборки мусора.
    ```

- **Soft Reference**

  объект с **Soft Reference** даже если свободен для сборки мусора, он всё равно не будет удалён сборщиком мусора, пока JVM остро не потребуется память. **Объекты удаляются из памяти, когда у JVM заканчивается память.**

    ```java
    // Если у тебя есть объект:
    Object obj = new Object();
    
    // И ты оборачиваешь его в SoftReference:
    SoftReference<Object> softRef = new SoftReference<>(obj);
    obj = null;
    
    // Получить объект можно так:
    Object value = softRef.get();
    
    // Если GC уже удалил объект → вернётся **null**
    ```

- **Weak Reference**

  `WeakReference` используется для объектов, которые должны быть удалены сразу, как только на них не остаётся strong-ссылок.

    ```java
    Object obj = new Object();          // strong reference
    WeakReference<Object> weak = new WeakReference<>(obj);
    
    obj = null; // убрали strong reference
    // осталась только weak-ссылка
    // объект становится кандидатом на удаление
    
    weak.get(); // может вернуть объект или null
    ```

  `WeakReference`   например используется в `WeakHashMap`

- **Phantom Reference**

  Это `НЕ ссылка` для работы с объектом, а `ДАТЧИК СМЕРТИ` объекта

  Используется для очистки всяких ресурсов, которые не контролируются GC.

    ```java
    // Шаг 1 — создаёшь очередь
    ReferenceQueue<MyObject> queue = new ReferenceQueue<>();
    
    // Шаг 2 — создаёшь PhantomReference
    PhantomReference<MyObject> ref = new PhantomReference<>(obj, queue);
    
    // Шаг 3 — убираешь strong reference
    obj = null;
    
    // Шаг 4 — GC делает магию:
    // Когда GC приходит:
    // 1) видит, что объект больше не reachable
    // 2) помечает его как **phantom reachable**
    // 3) кладёт ref в очередь
    
    // Шаг 5 — ты ловишь событие
    Reference<?> ref = queue.poll();
    if (ref != null) {
        // объект умер → чистим ресурсы
    }
    ```

  `phantomRef.get() == null` **ВСЕГДА NULL**!

  Класс **`ReferenceQueue`** предоставляет два метода для опроса очереди:
    - **`remove()`** - блокирует выполнение, когда очередь пустая и до тех пор, пока не в очереди не появится элемент для возращения
    -  **`poll()`** -  который не блокирует выполнение, и немедленно возвращает null, когда очередь пустая.