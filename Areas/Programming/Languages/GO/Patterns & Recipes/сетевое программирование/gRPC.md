
## Что такое gRPC

gRPC — это фреймворк для удалённого вызова процедур (RPC), разработанный Google.

Главная идея:

> Вызывать методы на удалённом сервере так, будто это обычные функции.

Работает поверх:

- [[HTTP2]]
    
- [[TCP]]
    
- использует [[Protocol Buffers]]
    

---

# 🧠 Что такое RPC

RPC (Remote Procedure Call) — это модель:

Клиент вызывает функцию → функция выполняется на сервере → возвращается результат

Ты пишешь:

user, err := client.GetUser(ctx, request)

Но на самом деле:

- создаётся сетевой запрос
    
- сериализуется в Protobuf
    
- отправляется по HTTP/2
    
- сервер выполняет метод
    
- отправляет ответ обратно
    

---

# 🧱 Как устроен gRPC

## 1️⃣ Описание сервиса в [[файл proto]]

syntax = "proto3";  
  
service UserService {  
  rpc GetUser (UserRequest) returns (UserResponse);  
}  
  
message UserRequest {  
  int32 id = 1;  
}  
  
message UserResponse {  
  int32 id = 1;  
  string name = 2;  
}

## 2️⃣ Генерация кода

protoc --go_out=. --go-grpc_out=. user.proto

Генерируются:

- структуры
- интерфейсы сервиса
- клиент

# 🧠 Что происходит после генерации

Из `.proto` генерируется:

##  [[server pb go]]

- структуры message
- методы сериализации
- дескрипторы схемы

##  [[server_grpc pb go]]

- интерфейс сервера
- клиентский stub
- регистрация методов
- обёртки вызовов
---
! для кореектной работы нужно докачать зависимости:
go get google.golang.org/protobuf@latest
go get google.golang.org/grpc@latest
## 3️⃣ Реализация сервера

type server struct {  
    pb.UnimplementedUserServiceServer  
}  
  
func (s *server) GetUser(ctx context.Context, req *pb.UserRequest) (*pb.UserResponse, error) {  
    return &pb.UserResponse{  
        Id:   req.Id,  
        Name: "Nazar",  
    }, nil  
}

---

## 4️⃣ Запуск сервера

lis, _ := net.Listen("tcp", ":50051")  
grpcServer := grpc.NewServer()  
pb.RegisterUserServiceServer(grpcServer, &server{})  
grpcServer.Serve(lis)

---

## 5️⃣ Клиент

conn, _ := grpc.Dial("localhost:50051", grpc.WithInsecure())  
client := pb.NewUserServiceClient(conn)  
  
resp, _ := client.GetUser(ctx, &pb.UserRequest{Id: 1})

---

# 🌐 Почему HTTP/2

gRPC использует [[HTTP2]], потому что он поддерживает:

- Multiplexing (несколько запросов в одном соединении)
    
- Streaming
    
- Более эффективную передачу данных
    

HTTP/1.1 не подходит для этого уровня производительности.

---

# 📡 Типы вызовов в gRPC

## 1️⃣ Unary (обычный запрос-ответ)

Классический RPC.

---

## 2️⃣ Server Streaming

Сервер отправляет поток данных.

---

## 3️⃣ Client Streaming

Клиент отправляет поток данных.

---

## 4️⃣ Bidirectional Streaming

Обе стороны обмениваются потоками.

Это мощнее, чем обычный [[REST API]].

---

# 🔥 gRPC vs REST

|REST|gRPC|
|---|---|
|HTTP/1.1|HTTP/2|
|JSON|Protobuf|
|Текст|Бинарный|
|Читаемый|Быстрый|
|Подходит для фронта|Подходит для микросервисов|

---

# 🧠 Когда использовать gRPC

Подходит для:

- микросервисной архитектуры
    
- внутреннего общения сервисов
    
- high-load систем
    
- real-time стриминга
    

Не очень подходит для:

- публичных браузерных API (без дополнительной настройки)
    
- простых CRUD-сервисов
    

---

# 🔐 Безопасность

Работает через:

- [[TLS]]
    
- mTLS
    

---

# 🔗 Связанные темы

- [[Protocol Buffers]]
    
- [[HTTP2]]
    
- [[TCP]]
    
- [[Socket]]
    
- [[REST API]]
    
- [[Context]]
    
- [[Microservices]]
    

---

# 📌 Главное понимание

gRPC — это:

> Строго типизированный, высокопроизводительный способ общения сервисов через удалённые вызовы функций.

Он скрывает сетевую сложность и даёт ощущение локального вызова.

---

# 💎 Инсайт для тебя

Когда ты пишешь:

client.GetUser(...)

Ты на самом деле запускаешь:

- сериализацию
    
- HTTP/2 фрейминг
    
- TCP передачу
    
- десериализацию
    

И всё это скрыто под красивым API.