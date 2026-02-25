# 🚀 `*_grpc.pb.go` — что это за файл и за что он отвечает

`*_grpc.pb.go` — это **RPC-обвязка**, сгенерированная из `.proto`.

Если коротко:

> Этот файл связывает твой `.proto` контракт с `grpc.Server` и `grpc.ClientConn`.

Он НЕ содержит бизнес-логики.  
Он содержит инфраструктуру вызова RPC.

---

# 📂 Что именно в нём лежит

В файле обычно 5 ключевых частей:

1. Константа полного имени сервиса
    
2. Интерфейс сервера
    
3. Unimplemented-заглушка
    
4. Регистрация сервиса в grpc.Server
    
5. Клиентский stub
    

Разберём каждую.

---

# 1️⃣ Полное имя сервиса (Service Descriptor)

Внутри есть структура вроде:

var GreeterService_ServiceDesc = grpc.ServiceDesc{  
    ServiceName: "greeter.v1.GreeterService",  
    HandlerType: (*GreeterServiceServer)(nil),  
    Methods: []grpc.MethodDesc{  
        {  
            MethodName: "SayHello",  
            Handler:    _GreeterService_SayHello_Handler,  
        },  
    },  
}

### Что это?

Это **таблица маршрутизации**.

Она говорит gRPC:

- если пришёл вызов
    
- с именем `/greeter.v1.GreeterService/SayHello`
    
- то нужно вызвать вот этот handler
    

Это аналог роутинга в HTTP.

---

# 2️⃣ Интерфейс сервера

type GreeterServiceServer interface {  
    SayHello(context.Context, *HelloRequest) (*HelloReply, error)  
    mustEmbedUnimplementedGreeterServiceServer()  
}

Это интерфейс, который ты должен реализовать.

Важно понимать:

- gRPC не знает про твой struct
    
- он работает через этот интерфейс
    
- твой сервер — это просто реализация этого интерфейса
    

---

# 3️⃣ UnimplementedGreeterServiceServer

type UnimplementedGreeterServiceServer struct{}

И метод:

func (UnimplementedGreeterServiceServer) SayHello(...) (...) {  
    return nil, status.Errorf(codes.Unimplemented, "method SayHello not implemented")  
}

### Зачем это нужно?

Чтобы:

- при добавлении нового RPC код не ломался
    
- сервер возвращал `Unimplemented`, если метод не реализован
    

Ты встраиваешь его:

type server struct {  
    greeterpb.UnimplementedGreeterServiceServer  
}

Это защита эволюции API.

---

# 4️⃣ Регистрация сервиса

func RegisterGreeterServiceServer(s grpc.ServiceRegistrar, srv GreeterServiceServer) {  
    s.RegisterService(&GreeterService_ServiceDesc, srv)  
}

Что происходит:

1. Ты передаёшь `grpcServer`
    
2. Передаёшь свою реализацию
    
3. gRPC добавляет в registry таблицу маршрутизации
    

После этого сервер знает:

- какой сервис существует
    
- какие методы он поддерживает
    
- какую функцию вызывать
    

---

# 5️⃣ Handler-функция (самое важное)

Будет что-то вроде:

func _GreeterService_SayHello_Handler(  
    srv interface{},  
    ctx context.Context,  
    dec func(interface{}) error,  
    interceptor grpc.UnaryServerInterceptor,  
) (interface{}, error)

Вот здесь магия раскрывается.

Разберём шаги.

### 1. Создаётся пустой объект запроса

in := new(HelloRequest)

### 2. Вызов dec()

if err := dec(in); err != nil {  
    return nil, err  
}

`dec` — это функция, которая:

- берёт байты из HTTP/2
    
- десериализует protobuf
    
- заполняет `in`
    

### 3. Если нет interceptor

return srv.(GreeterServiceServer).SayHello(ctx, in)

То есть просто вызывается твой метод.

### 4. Если есть interceptor

Создаётся обёртка:

handler := func(ctx context.Context, req interface{}) (interface{}, error) {  
    return srv.(GreeterServiceServer).SayHello(ctx, req.(*HelloRequest))  
}

И вызывается interceptor.

---

# 🧠 Что это значит инженерно

`*_grpc.pb.go` — это:

- декодирование входящих protobuf сообщений
    
- вызов твоей функции
    
- обработка interceptor'ов
    
- упаковка ответа обратно в protobuf
    
- передача в grpc runtime
    

Это glue-код между:

- transport (HTTP/2)
    
- protobuf
    
- твоей логикой
    

---

# 6️⃣ Клиентская часть

В файле также будет:

type GreeterServiceClient interface {  
    SayHello(ctx context.Context, in *HelloRequest, opts ...grpc.CallOption) (*HelloReply, error)  
}

И реализация:

func (c *greeterServiceClient) SayHello(...) (...) {  
    err := c.cc.Invoke(ctx, "/greeter.v1.GreeterService/SayHello", in, out, opts...)  
}

Что делает `Invoke`:

1. сериализует `in` в protobuf
    
2. создаёт HTTP/2 stream
    
3. отправляет данные
    
4. ждёт ответ
    
5. десериализует в `out`
    

---

# 🔎 Как это связано с твоим `main.go`

Когда ты пишешь:

grpcServer := grpc.NewServer()  
greeterpb.RegisterGreeterServiceServer(grpcServer, &server{})  
grpcServer.Serve(lis)

Происходит:

- регистрируется `ServiceDesc`
    
- при запросе вызывается `_GreeterService_SayHello_Handler`
    
- он вызывает твой `server.SayHello`
    

---

# 📌 Краткая модель

`greeter.proto`  
↓  
`*_grpc.pb.go`  
↓  
grpc.Server  
↓  
HTTP/2  
↓  
protobuf  
↓  
твоя функция

---

# ⚙ Главное понимание

Этот файл:

- не содержит бизнес-логики
    
- не содержит транспортного кода
    
- он соединяет контракт и runtime
    

Он превращает:

client.SayHello(...)

в:

HTTP/2 stream  
+ protobuf  
+ routing  
+ handler  
+ interceptor