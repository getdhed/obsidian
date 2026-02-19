
## Что такое Pointer

**Pointer** — это переменная, которая хранит **адрес другой переменной в памяти**.

Он не хранит значение напрямую — он хранит ссылку на него.

value  →  10  
pointer →  0x1400012a

---

## Pointer в Go

В Go указатель объявляется через `*`.

var x int = 10  
var p *int = &x

- `&x` → взять адрес переменной
    
- `*p` → получить значение по адресу
    

---

## Разыменование

x := 5  
p := &x  
  
fmt.Println(*p) // 5

`*p` — это разыменование (dereference).

Если изменить через pointer:

*p = 20  
fmt.Println(x) // 20

Мы изменили оригинальную переменную.

---

## Зачем нужны Pointers

### 1. Изменение значения в функции

Без pointer:

func inc(x int) {  
    x++  
}

С pointer:

func inc(x *int) {  
    (*x)++  
}

Использование:

n := 10  
inc(&n)

---

### 2. Избежание копирования

Структуры копируются по значению.

type User struct {  
    Name string  
    Age  int  
}

Если структура большая — копирование дорого.

Поэтому:

func process(u *User)

Связано:

- [[Struct]]
    
- [[Method]]
    
- [[Memory]]
    

---

## Pointer и Struct

Очень частая практика:

u := &User{  
    Name: "Bob",  
    Age:  30,  
}

Go автоматически разыменует:

u.Name = "Alice"

Хотя `u` — pointer.

---

## Pointer Receiver

Методы могут иметь pointer receiver.

func (u *User) UpdateName(name string) {  
    u.Name = name  
}

Почему pointer?

- изменяет структуру
    
- не копирует её
    
- эффективнее
    

Связано:

- [[Method]]
    
- [[Struct]]
    
- [[OOP in Go]]
    

---

## Nil Pointer

Pointer может быть `nil`.

var p *int  
fmt.Println(p) // nil

Если разыменовать:

fmt.Println(*p) // panic

Это вызовет runtime panic.

Связано:

- [[Nil]]
    
- [[Panic]]
    
- [[Error Handling]]
    

---

## new vs &

### new

p := new(int)

- выделяет память
    
- возвращает pointer
    

### &

x := 10  
p := &x

Обычно `&` используется чаще.

---

## Pointer и Heap

Go сам решает:

- stack
    
- heap
    

Если pointer "убегает" из функции → escape analysis → heap.

Связано:

- [[Memory]]
    
- [[Garbage Collector]]
    
- [[Escape Analysis]]
    

---

## Pointer и Concurrency

Если несколько goroutine используют pointer к одной структуре:

→ возможна [[Race Condition]]

Пример:

go func() {  
    user.Name = "A"  
}()

Нужно использовать:

- [[Mutex]]
    
- [[Atomic]]
    
- [[Channel]]
    

---

## Pointer и JSON

Очень часто в API используются pointer поля:

type User struct {  
    Name *string `json:"name,omitempty"`  
}

Зачем?

- отличить "не передано"
    
- от "передано пустое значение"
    

Связано:

- [[JSON]]
    
- [[Serialization]]
    
- [[REST API]]
    

---

## Pointer vs Value

Когда использовать pointer:

✔ структура большая  
✔ нужно изменить значение  
✔ метод должен менять состояние  
✔ нужно избежать копирования

Когда не нужно:

✘ маленькие структуры  
✘ immutable данные  
✘ примитивы без необходимости изменения

---

## Pointer и Interface

Интерфейсы в Go уже содержат:

- type
    
- value
    

Если положить pointer в interface — внутри будет pointer.

Связано:

- [[Interfaces]]
    
- [[Polymorphism]]
    

---

## Отличие от C/C++

В Go:

- нет арифметики указателей
    
- нет pointer math
    
- нет manual free
    
- есть [[Garbage Collector]]
    

Это делает Go безопаснее.

---

## Главное понимание

Pointer в Go — это:

- механизм работы с памятью
    
- способ избежать копирования
    
- способ менять состояние
    
- фундамент методов и архитектуры
    

Он важен для:

- [[Struct]]
    
- [[Method]]
    
- [[Concurrency]]
    
- [[API]]
    
- [[Microservices]]