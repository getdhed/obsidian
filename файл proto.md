
# 📦 `.proto` файл — контракт gRPC сервиса

`.proto` — это **IDL (Interface Definition Language)** для описания:

- структуры данных (messages)
- RPC методов (service)
- версий
- пакетов
- опций генерации кода

Это не код выполнения.  
Это **контракт**, из которого генерируется код для клиента и сервера.

---

# 🔤 `syntax`

syntax = "proto3";

Определяет правила языка.

В `proto3`:

- нет обязательных полей (все optional по сути)
- нулевые значения считаются "не задано"
- сериализация компактная

---

# 📦 `package`

package greeter.v1;

Это **protobuf namespace**.

Он влияет на:

- полное имя сервиса на wire-уровне
- имена сообщений внутри протокола

Полное имя RPC будет выглядеть так:

/greeter.v1.GreeterService/SayHello

Важно:

- это не Go-пакет
- это имя в протоколе

---

# ⚙ `option go_package`

option go_package = "grpc-hello/gen/greeter/v1;greeterpb";

Говорит генератору Go:

1. куда класть код
2. какое имя пакета использовать

Формат:

"import/path;packageName"

---

# 🧱 `message`

message HelloRequest {  
    string name = 1;  
}

## Что такое message

`message` — это структура данных, которая:

- сериализуется в бинарный protobuf
- передаётся по сети
- может использоваться в RPC

Это аналог:

type HelloRequest struct {  
    Name string  
}

---

## Поля в message

string name = 1;

Разберём по частям:

- `string` — тип
    
- `name` — имя поля
    
- `= 1` — **номер поля в бинарном протоколе**
    

### Почему номер важен?

В protobuf на проводе передаётся:

[tag number] + [type] + [value]

Имя поля **в сеть не уходит**.

Поэтому:

- менять имя безопасно
    
- менять номер нельзя (ломает совместимость)
    

---

## Базовые типы

- string
    
- int32 / int64
    
- bool
    
- float / double
    
- bytes
    

---

## Повторяющиеся поля

repeated string tags = 2;

→ в Go будет `[]string`

---

## Вложенные message

message User {  
    string name = 1;  
}  
  
message Response {  
    User user = 1;  
}

Message могут содержать другие message.

---

# 🚀 `service`

service GreeterService {  
    rpc SayHello (HelloRequest) returns (HelloReply);  
}

## Что такое service

`service` — это набор RPC методов.

Это аналог интерфейса в Go:

type GreeterService interface {  
    SayHello(ctx context.Context, req *HelloRequest) (*HelloReply, error)  
}

---

## `rpc`

rpc SayHello (HelloRequest) returns (HelloReply);

Это объявление удалённого метода.

Оно говорит:

- метод называется `SayHello`
    
- принимает `HelloRequest`
    
- возвращает `HelloReply`
    

Ничего не запускается.  
Это только описание.

---

# 🔢 Почему поля нумеруются

string name = 1;

Это нужно для:

- компактной сериализации
    
- совместимости версий
    

Правила:

- нельзя менять номер существующего поля
    
- можно добавлять новые поля с новыми номерами
    
- удалённые номера лучше не переиспользовать
    

---

# 🧠 Что происходит после генерации

Из `.proto` генерируется:

## 1️⃣ `*.pb.go`

- структуры message
- методы сериализации
- дескрипторы схемы

## 2️⃣ `*_grpc.pb.go`

- интерфейс сервера
- клиентский stub
- регистрация методов
- обёртки вызовов

---

# 🧩 Инженерная модель

`.proto` = контракт

- message → тип данных
    
- service → интерфейс RPC
    
- rpc → метод
    
- package → namespace протокола
    
- go_package → куда генерировать код
    

Клиент и сервер **обязаны использовать один и тот же контракт**.