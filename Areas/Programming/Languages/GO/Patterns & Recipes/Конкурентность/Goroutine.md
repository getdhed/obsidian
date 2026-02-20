
## Что такое Goroutine

**Goroutine** — это лёгкий поток выполнения в Go.

Создаётся через ключевое слово `go`:

go func() {  
    fmt.Println("hello")  
}()

Это основа [[Concurrency]] в Go.

---

## Главное отличие от Thread

Goroutine ≠ OS thread.

|Goroutine|Thread|
|---|---|
|управляется Go runtime|управляется ОС|
|дешёвая|дорогая|
|тысячи / миллионы|ограничено|

Go использует модель [[GMP Model]]:

- G — goroutine
    
- M — OS thread
    
- P — processor
    

Связано:

- [[Scheduler]]
    
- [[GMP Model]]
    

---

## Как создаётся

func sayHello() {  
    fmt.Println("hi")  
}  
  
go sayHello()

Важно:

Если `main()` завершится — программа завершится, даже если goroutine ещё работает.

---

## Параллельность

Goroutine даёт **concurrency**, но не всегда parallelism.

Настоящий parallelism зависит от:

runtime.GOMAXPROCS()

Связано:

- [[Concurrency]]
    
- [[Parallelism]]
    

---

## Анонимные goroutine

Очень частый паттерн:

go func(name string) {  
    fmt.Println(name)  
}("Bob")

---

## Передача данных

Goroutine не должна работать с shared memory без синхронизации.

Правильный способ — [[Channel]]:

ch := make(chan int)  
  
go func() {  
    ch <- 10  
}()  
  
val := <-ch

Связано:

- [[Channel]]
    
- [[Select]]
    

---

## WaitGroup

Чтобы дождаться завершения goroutine:

var wg sync.WaitGroup  
  
wg.Add(1)  
go func() {  
    defer wg.Done()  
}()  
  
wg.Wait()

Связано:

- [[WaitGroup]]
    
- [[Synchronization]]
    

---

## Goroutine и Loop Variable Bug

Частая ошибка:

for i := 0; i < 5; i++ {  
    go func() {  
        fmt.Println(i)  
    }()  
}

Проблема — захватывается одна и та же переменная.

Правильно:

for i := 0; i < 5; i++ {  
    i := i  
    go func() {  
        fmt.Println(i)  
    }()  
}

Связано:

- [[Closure]]
    
- [[Race Condition]]
    

---

## Race Condition

Если несколько goroutine меняют одну переменную:

go func() {  
    counter++  
}()

Нужен:

- [[Mutex]]
    
- [[Atomic]]
    
- [[Channel]]
    

Проверка:

go run -race main.go

---

## Goroutine и Context

Для управления временем жизни:

ctx, cancel := context.WithCancel(context.Background())  
  
go func() {  
    select {  
    case <-ctx.Done():  
        return  
    }  
}()

Очень важно в:

- [[net/HTTP]]
    
- [[gRPC]]
    
- [[Microservices]]
    

---

## Goroutine в HTTP Server

Каждый запрос обрабатывается в отдельной goroutine:

http.HandleFunc("/", handler)  
http.ListenAndServe(":8080", nil)

Под капотом:

- connection → goroutine
    
- request → goroutine
    

Связано:

- [[net/HTTP]]
    
- [[Server]]
    
- [[TCP]]
    

---

## Stack

Goroutine имеет свой stack.

Особенность:

- стартует с маленького размера (~2KB)
    
- автоматически растёт
    
- автоматически уменьшается
    

В отличие от OS thread — фиксированный stack.

---

## Планировщик (Scheduler)

Go runtime:

- создаёт M threads
    
- распределяет goroutine
    
- переключает их (cooperative scheduling)
    

Goroutine может быть:

- runnable
    
- running
    
- blocked
    

Связано:

- [[Scheduler]]
    
- [[GMP Model]]
    

---

## Блокировка Goroutine

Goroutine блокируется при:

- ожидании канала
    
- системном вызове
    
- ожидании mutex
    
- чтении из сети
    

Например:

val := <-ch

Если канал пуст — goroutine ждёт.

---

## Deadlock

Если все goroutine заблокированы:

fatal error: all goroutines are asleep - deadlock!

Связано:

- [[Deadlock]]
    
- [[Channel]]
    

---

## Паттерны

Популярные паттерны:

- Worker Pool
    
- Fan-out
    
- Fan-in
    
- Pipeline
    
- Producer / Consumer
    

Связано:

- [[Channel]]
    
- [[Select]]
    
- [[Concurrency Patterns]]
    

---

## Goroutine и Microservices

Goroutine лежит в основе:

- [[HTTP Server]]
    
- [[gRPC]]
    
- [[REST API]]
    
- [[Microservices]]
    

Без неё сервер обрабатывал бы один запрос за раз.

---

## Главное понимание

Goroutine — это:

- лёгкий поток выполнения
    
- управляется runtime
    
- масштабируется тысячами
    
- основа сетевого backend-кода
    

Она делает Go:

- простым для concurrent кода
    
- эффективным
    
- подходящим для high-load систем