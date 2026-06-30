# Cleaners

Начиная с Java 9, Cleaner - автоматически выполняет cleanup (очистку ресурсов), когда объект становится недостижимым

По сути:

- это **обёртка над `PhantomReference` + `ReferenceQueue`**

`Cleaner` используются объекты `PhantomReference` и `ReferenceQueue` для отслеживания зарегистрированных объектов. Как только объект становится **`«phantom reachable»,`** `ReferenceQueue` уведомляет `Cleaner` и тот использует свой поток для выполнения соответствующего действия по очистке.

Самый простой Cleaner:

```java
import java.lang.ref.Cleaner;

public class ResourceHolder {
 private static final Cleaner CLEANER = Cleaner.create();
     public ResourceHolder() {
         CLEANER.register(this, () -> System.out.println("doing some clean up"));
     }
     public static void main(String... args) {
         ResourceHolder resourceHolder = new ResourceHolder();
         resourceHolder = null;
         System.gc();
     }
}
```

Очень сильно рекомендуется **не использовать** `lambda` и уж тем более так:

Обращаясь к **`cleaningMessage`** в лямбда, мы заставляем её хранить ссылку на этот объект `this` (`ResourceHolder`). Благодаря этому объект никогда не станет `phantom reachable`, поскольку код нашего приложения всё ещё будет иметь на него ссылку.

<aside>
💡

`lambda` не будут неявно хранить ссылку на содержащий объект (в отличии от `inner class`*)*, если только их к этому не принудят. То есть 1-ый пример рабочий, а 2-ой - НЕТ, так как там есть `instance variable`

</aside>

<aside>
💡

`Cleaner` не должен создаваться на каждый объект — его нужно переиспользовать (обычно один на всё приложение). Так как один `Cleaner` использует один `Java thread` (платформенный поток).

</aside>

Правильное использование:

```java
import java.lang.ref.Cleaner;

public class ResourceHolder {

    private final Cleaner.Cleanable cleanable;
    private final ExternalResource externalResource;

    public ResourceHolder(ExternalResource externalResource) {
        cleanable = AppUtil.getCleaner()
										       .register(this, new CleaningAction(externalResource));
        this.externalResource = externalResource;
    }

//    You can call this method whenever is the right time to release resource
    public void releaseResource() {
        cleanable.clean();
    }

    public void doSomethingWithResource() {
        System.out.printf("Do smth cool with the resource: %s \n", 
									        this.externalResource);
    }

    static class CleaningAction implements Runnable {
        private ExternalResource externalResource;

        CleaningAction(ExternalResource externalResource) {
            this.externalResource = externalResource;
        }

        @Override
        public void run() {
//          Cleaning up the important resources
            System.out.println("releasing up very important resource");
            externalResource = null;
        }
    }

    public static void main(String... args) {
        ResourceHolder resourceHolder = new ResourceHolder(new ExternalResource());
        resourceHolder.doSomethingWithResource();
/*
        After doing some important work, we can explicitly release
        resources/invoke the cleaning action
*/
        resourceHolder.releaseResource();
//      What if we explicitly invoke the cleaning action twice?
        resourceHolder.releaseResource();
    }
}

```

Самое эффективное использование с **`AutoCloseable` :**

```java
import java.lang.ref.Cleaner.Cleanable;

public class ResourceHolder implements AutoCloseable {

    private final ExternalResource externalResource;

    private final Cleaner.Cleanable cleanable;

    public ResourceHolder(ExternalResource externalResource) {
        this.externalResource = externalResource;
        cleanable = AppUtil.getCleaner().register(this, new CleaningAction(externalResource));
    }

    public void doSomethingWithResource() {
        System.out.printf("Do something cool with the important resource: %s \n", this.externalResource);
    }
    @Override
    public void close() {
        System.out.println("cleaning action invoked by the close method");
        cleanable.clean();
    }

    static class CleaningAction implements Runnable {
        private ExternalResource externalResource;

        CleaningAction(ExternalResource externalResource) {
            this.externalResource = externalResource;
        }

        @Override
        public void run() {
//            cleaning up the important resources
            System.out.println("Doing some cleaning logic here, releasing up very important resources");
            externalResource = null;
        }
    }

    public static void main(String[] args) {
//      This is an effective way to use cleaners with instances that hold crucial resources
        try (ResourceHolder resourceHolder = new ResourceHolder(new ExternalResource(1))) {
            resourceHolder.doSomethingWithResource();
            System.out.println("Goodbye");
        }
/*
    In case the client code does not use the try-with-resource as expected,
    the Cleaner will act as the safety-net
*/
        ResourceHolder resourceHolder = new ResourceHolder(new ExternalResource(2));
        resourceHolder.doSomethingWithResource();
        resourceHolder = null;
        System.gc(); // to facilitate the running of the cleaning action
    }
}
```

**`Cleaner`**

Он нужен, когда:

- у тебя есть **внешний ресурс** (файл, сокет, native память)
- и ты хочешь гарантировать его очистку, даже если разработчик забыл вызвать `close()`

### Основные сущности:

`Cleanable`  - Это просто "обёртка" вокруг cleanup-действия

`PhantomCleanable` -  это внутренняя реализация:

- наследуется от `PhantomReference`
- связывает:
    - объект (referent)
    - cleanup-действие

### Жизненный цикл `Cleaner`

- **`Cleaner.create()`**

  ### 1. Создаётся `ReferenceQueue`

    ```java
    ReferenceQueue<Object>queue=newReferenceQueue<>();
    ```

  👉 Это очередь, куда GC будет класть "мертвые" объекты
    
  ---

  ### 2. Создаётся список `PhantomCleanable`

  Cleaner хранит:

    ```java
    LinkedList<PhantomCleanable>
    ```

  👉 зачем?

  Чтобы:

    - отслеживать все зарегистрированные cleanup-и
    - контролировать lifecycle

    ---

  ### 3. Cleaner регистрирует сам себя

  Это хитрый момент 👇

  Cleaner создаёт `PhantomCleanable` **для самого себя**

  👉 чтобы:

    - поток Cleaner не завершился раньше времени

    ---

  ### 4. Запускается фоновый thread

    ```java
    newThread(() -> {
    while (!list.isEmpty()) {
    queue.remove();// блокируется
        }
    }).start();
    ```

  👉 этот поток:

    - ждёт, пока GC добавит объект в очередь
    - потом вызывает cleanup

- **`cleaner.register()`**

    ```java
    cleaner.register(resourceHolder, cleaningAction);
    ```

  Что происходит:

    1. создаётся `PhantomCleanable`:

    ```
    referent = resourceHolder
    action = cleaningAction
    ```

    1. добавляется в `linked list`

- **GC убивает объект**

    ```java
    resourceHolder = null;
    System.gc();
    ```

  GC делает:

  👉 объект становится **`phantom reachable`**

  👉 создаётся событие:

    ```
    PhantomCleanable → попадает в ReferenceQueue
    ```

- **Cleaner thread реагирует**

  Поток делает:

    ```java
    PhantomCleanableref=queue.remove();
    ref.clean();
    ```