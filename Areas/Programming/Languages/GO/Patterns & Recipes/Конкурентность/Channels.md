
## Что такое Channel

**Channel** — это механизм передачи данных между [[Areas/Programming/Languages/GO/Patterns & Recipes/Конкурентность/Goroutine]].

Он позволяет:

- синхронизировать выполнение
    
- безопасно передавать данные
    
- избегать shared memory
    

Go следует принципу:

> Don’t share memory by communicating; communicate by sharing memory

Связано:

- [[Concurrency]]
    
- [[Areas/Programming/Languages/GO/Patterns & Recipes/Конкурентность/Goroutine]]
    

---

## Создание канала

ch := make(chan int)

Тип канала всегда указывается:

chan int  
chan string  
chan *User

Связано:

- [[Pointers]]
    
- [[Struct]]
    

---

## Отправка и получение

### Отправка

ch <- 10

### Получение

val := <-ch

Операции блокирующие.

Если:

- нет получателя → отправка блокируется
    
- канал пуст → чтение блокируется
    

---

## Unbuffered Channel

ch := make(chan int)

Особенность:

- отправка блокируется до чтения
    
- чтение блокируется до отправки
    

Это создаёт синхронизацию.

Пример:

go func() {  
    ch <- 5  
}()  
  
val := <-ch

---

## Buffered Channel

ch := make(chan int, 3)

Буфер позволяет:

- отправить N значений без немедленного чтения
    

ch <- 1  
ch <- 2

Блокировка произойдёт, когда буфер заполнится.

---

## Закрытие канала

close(ch)

Закрывать должен тот, кто отправляет данные.

После закрытия:

- можно читать оставшиеся значения
    
- отправка вызовет panic
    

val, ok := <-ch

`ok == false` → канал закрыт.

Связано:

- [[Nil]]
    
- [[Panic]]
    

---

## Range по каналу

for v := range ch {  
    fmt.Println(v)  
}

Цикл завершится, когда канал закрыт.

---

## Select

Позволяет ожидать несколько каналов.

select {  
case v := <-ch1:  
    fmt.Println(v)  
case <-ch2:  
    fmt.Println("done")  
}

Используется для:

- таймаутов
    
- cancellation
    
- multiplexing
    

Связано:

- [[Select]]
    
- [[Context]]
    

---

## Default в Select

select {  
case v := <-ch:  
    fmt.Println(v)  
default:  
    fmt.Println("no data")  
}

Позволяет сделать non-blocking проверку.

---

## Channel Directions

Можно ограничить направление:

func sendOnly(ch chan<- int)  
func recvOnly(ch <-chan int)

Это повышает безопасность API.

Связано:

- [[API]]
    
- [[Clean Architecture]]
    

---

## Nil Channel

var ch chan int

Nil канал:

- чтение → блокируется навсегда
    
- запись → блокируется навсегда
    

Используется иногда для динамического отключения case в select.

---

## Deadlock

Если все goroutine ждут друг друга:

fatal error: all goroutines are asleep - deadlock!

Пример:

ch := make(chan int)  
ch <- 1

Связано:

- [[Deadlock]]
    
- [[Areas/Programming/Languages/GO/Patterns & Recipes/Конкурентность/Goroutine]]
    

---

## Channel и Race Condition

Channel помогает избежать:

- shared memory
    
- [[Race Condition]]
    

Но не защищает автоматически от логических ошибок.

---

## Channel и Context

Частый паттерн:

select {  
case <-ctx.Done():  
    return  
case v := <-ch:  
    fmt.Println(v)  
}

Очень важно в:

- [[net/HTTP]]
    
- [[gRPC]]
    
- [[Microservices]]
    

---

## Worker Pool

Классический пример:

jobs := make(chan int)  
results := make(chan int)  
  
go worker(jobs, results)

Связано:

- [[Concurrency Patterns]]
    
- [[WaitGroup]]
    

---

## Fan-out / Fan-in

Fan-out — несколько goroutine читают из одного канала.  
Fan-in — объединение нескольких каналов в один.

Связано:

- [[Select]]
    
- [[Areas/Programming/Languages/GO/Patterns & Recipes/Конкурентность/Goroutine]]
    

---

## Channel и Network

Channel часто используется внутри:

- [[HTTP Server]]
    
- [[TCP]]
    
- [[Socket]]
    
- [[Server]]
    

Например:

- очередь задач
    
- обработка запросов
    
- event loop
    

---

## Когда использовать Channel

✔ передача данных между goroutine  
✔ синхронизация  
✔ event-driven логика  
✔ pipeline

Не использовать:

✘ как замену обычной функции  
✘ без необходимости concurrency

---

## Channel vs Mutex

|Channel|Mutex|
|---|---|
|передача данных|защита памяти|
|message passing|shared memory|
|подходит для pipeline|подходит для state|

Связано:

- [[Mutex]]
    
- [[Atomic]]
    

---

## Главное понимание

Channel — это:

- механизм синхронизации
    
- способ коммуникации
    
- инструмент построения concurrent систем
    
- фундамент сетевого backend-кода
    

Он лежит в основе:

- [[Areas/Programming/Languages/GO/Patterns & Recipes/Конкурентность/Goroutine]]
    
- [[net/HTTP]]
    
- [[Microservices]]
    
- [[Concurrency]]