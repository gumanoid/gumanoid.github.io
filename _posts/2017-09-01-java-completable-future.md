Уже не помню, при каких обстоятельствах я впервые заинтересовался классом java.util.concurrent.CompletableFuture. Помню свою первую реакцию.

TODO Гифка. Скриншот Идеи с текстом 'new CompletableFuture()', нажимается точка, выпрыгивают десятки методов класса. Соединить с гифкой, где кто-нибудь открывает изнутри дома входную дверь, и в него вваливается сугроб.

Первая мысль при виде полусотни методов: "С названием класса ошиблись. Этот Future не Completable, он какой-то Complicated". И на какое-то время я забыл про этот класс.

Со второго захода разобраться удалось. С другими классами это получается, как правило, за счёт понимания, что это такое и зачем нужно. В случае CompletableFuture просветление настигло в тот момент, когда я осознал, что среди этих десятков методов много почти одинаковых, они логически группируются в небольшой набор операций. Практически каждая операция принимает на вход лямбду и возвращает CompletableFuture. Одной операции может соответствовать 12 (!) методов: 4 варианта лямбды и 3 варианта асинхронности. Потратив чуть-чуть времени на то, чтобы запомнить схему именования методов, можно существенно облегчить себе понимание.

CompletableFuture - это способ разделить два аспекта: *что* вычисляется и *кем* (в каком потоке) вычисляется. Для примера возьмём вот такой код:

    try {
        Type1 var1 = computeFirstStage();
        Type2 var2 = computeSecondStage(var1);
        Type3 var3 = computeThirdStage(var1, var2);
        System.out.println(var3);
    } catch (Exception error) {
        error.printStackTrace();
    }

Тот же код с использованием CompletableFuture выглядит так:

    CompletableFuture<Type1> future1 = CompletableFuture.supplyAsync(() -> computeFirstStage());
    CompletableFuture<Type2> future2 = future1.thenApply(var1 -> computeSecondStage(var1));
    CompletableFuture<Type3> future3 = future1.thenCombine(future2, (var1, var2) -> computeThirdStage(var1, var2));
    CompletableFuture<Type3> future4 = future3.whenComplete((value, error) -> {
        if (error == null) {
            System.out.println(value);
        } else {
            error.printStackTrace();
        }
    });

:
- создание экземпляра CompletableFuture
- преобразоване одного экземпляра в другой
- преобразование двух и более экземпляров в один
- указание реакции на завершение (получение значения/ошибки)
- вспомогательные методы


CompletableFuture<T> - контейнер на одно значение, который может находиться в одном из трёх состояний:
- пусто
- содержит T
- содержит Throwable

Возможен переход из состояния "пусто" в состояние "содержит T" или "содержит Throwable", и только (есть нюанс, но о нём позже).



-- Function
-- Consumer
-- Runnable
-- без постфикса Async
-- *Async с явным указанием Executor'а
-- *Async без указания Executor'а


