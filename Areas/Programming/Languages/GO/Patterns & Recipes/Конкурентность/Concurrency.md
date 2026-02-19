# Concurrency

## Что такое Concurrency

**Concurrency** — это способность программы выполнять несколько задач _одновременно по логике_, но не обязательно физически параллельно.

Важно не путать:

- **Concurrency** → про структуру программы
    
- **Parallelism** → про реальное одновременное выполнение на нескольких CPU
    

> Concurrency — это про управление множеством задач  
> Parallelism — это про одновременное выполнение

---

## Concurrency в Go

В Go конкурентность встроена в язык.

Основные инструменты:

- [[Areas/Programming/Languages/GO/Patterns & Recipes/Конкурентность/Goroutine]]
    
- [[Channel]]
    
- [[Select]]
    
- [[Mutex]]
    
- [[Context]]
    
- [[WaitGroup]]
    

Go использует модель:

> **CSP (Communicating Sequential Processes)**  
> "Don't communicate by sharing memory; share memory by communicating"

---

## Goroutine

`goroutine` — лёгкий поток выполнения.

go func() {  
    fmt.Println("hello")  
}()

Особенности:

- создаётся через `go`
    
- управляется рантаймом Go
    
- multiplexing поверх OS threads
    
- дешёвая (тысячи/миллионы)
    

Связанные темы:

- [[Areas/Programming/Languages/GO/Patterns & Recipes/Конкурентность/Goroutine]]
    
- [[Scheduler]]
    
- [[GMP Model]]
    

---

## Channels

`channel` — способ безопасной передачи данных между goroutine.

ch := make(chan int)  
  
go func() {  
    ch <- 10  
}()  
  
val := <-ch

Типы:

- unbuffered
    
- buffered
    
- closed channel
    

Связанные темы:

- [[Channel]]
    
- [[Select]]
    
- [[Deadlock]]
    

---

## Select

`select` — ожидание нескольких каналов.

select {  
case v := <-ch1:  
    fmt.Println(v)  
case <-ctx.Done():  
    return  
}

Связано с:

- [[Context]]
    
- [[Cancellation]]
    
- [[Timeout]]
    

---

## Synchronization

Когда используется shared memory — нужны примитивы синхронизации.

### Mutex

var mu sync.Mutex  
  
mu.Lock()  
counter++  
mu.Unlock()

### RWMutex

- RLock()
    
- Lock()
    

### WaitGroup

var wg sync.WaitGroup  
  
wg.Add(1)  
go func() {  
    defer wg.Done()  
}()  
  
wg.Wait()

Связанные темы:

- [[Mutex]]
    
- [[Race Condition]]
    
- [[Atomic]]
    

---

## Context

`context.Context` управляет:

- отменой операций
    
- дедлайнами
    
- таймаутами
    
- передачей request-scoped данных
    

Очень важен в:

- [[HTTP]]
    
- [[gRPC]]
    
- [[Microservices]]
    

---

## Concurrency в сетевом программировании

Каждый HTTP request обрабатывается в отдельной goroutine:

http.HandleFunc("/", handler)  
http.ListenAndServe(":8080", nil)

Под капотом:

- connection → goroutine
    
- request → goroutine
    

Связано с:

- [[HTTP]]
    
- [[TCP]]
    
- [[Socket]]
    
- [[Server]]
    

---

## Race Condition

Возникает, когда несколько goroutine:

- читают/пишут одну переменную
    
- без синхронизации
    

Проверяется:

go run -race main.go

Связано:

- [[Mutex]]
    
- [[Atomic]]
    
- [[Deadlock]]
    

---

## Deadlock

Когда goroutine:

- ждут друг друга
    
- никто не может продолжить выполнение
    

Пример:

ch := make(chan int)  
ch <- 1 // deadlock

---

## Concurrency Patterns

Частые паттерны:

- Worker Pool
    
- Fan-in / Fan-out
    
- Pipeline
    
- Producer / Consumer
    

Связано:

- [[Channel]]
    
- [[Select]]
    
- [[Areas/Programming/Languages/GO/Patterns & Recipes/Конкурентность/Goroutine]]
    

---

## GMP Model

Go использует модель:

- G — goroutine
    
- M — OS thread
    
- P — processor (logical)
    

Связано:

- [[Scheduler]]
    
- [[GMP Model]]
    

---

## Concurrency vs Network

Concurrency — фундамент для:

- [[HTTP Server]]
    
- [[gRPC]]
    
- [[Microservices]]
    
- [[API]]
    
- [[REST API]]
    

Без конкурентности сервер бы обрабатывал 1 запрос за раз.

---

## Главное понимание

Concurrency в Go — это:

- управление задачами
    
- координация выполнения
    
- безопасный обмен данными
    
- контроль отмены
    

Это основа:

- сетевого кода
    
- backend разработки
    
- микросервисов
    
- высоконагруженных систем