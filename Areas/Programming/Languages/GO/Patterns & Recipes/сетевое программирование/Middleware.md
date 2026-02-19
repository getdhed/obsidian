# 🧩 Middleware

## Что такое Middleware

Middleware — это промежуточный слой обработки запроса между клиентом и основным обработчиком (handler).

Главная идея:

> Выполнить дополнительную логику до или после основного обработчика.

Пример задач middleware:

- логирование
    
- авторизация
    
- аутентификация
    
- ограничение запросов (rate limit)
    
- обработка ошибок
    
- метрики
    

---

# 🧠 Где используется

- [[REST API]]
    
- [[gRPC]]
    
- веб-серверы
    
- микросервисы
    

---

# 🏗 Как работает Middleware в Go (net/http)

В Go middleware — это функция, которая принимает `http.Handler` и возвращает новый `http.Handler`.

Сигнатура:

func(next http.Handler) http.Handler

---

# 📌 Базовый пример

## Логирующий middleware

package main  
  
import (  
	"log"  
	"net/http"  
	"time"  
)  
  
func LoggingMiddleware(next http.Handler) http.Handler {  
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {  
		start := time.Now()  
  
		log.Println("Started:", r.Method, r.URL.Path)  
  
		next.ServeHTTP(w, r)  
  
		log.Println("Completed in:", time.Since(start))  
	})  
}  
  
func HelloHandler(w http.ResponseWriter, r *http.Request) {  
	w.Write([]byte("Hello World"))  
}  
  
func main() {  
	handler := LoggingMiddleware(http.HandlerFunc(HelloHandler))  
  
	http.ListenAndServe(":8080", handler)  
}

---

# 🔁 Как это работает

1. Приходит HTTP-запрос
    
2. Сначала вызывается middleware
    
3. Middleware вызывает `next.ServeHTTP`
    
4. Управление возвращается обратно
    

Это похоже на "обёртку".

---

# 🔐 Middleware для авторизации

func AuthMiddleware(next http.Handler) http.Handler {  
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {  
		token := r.Header.Get("Authorization")  
  
		if token != "secret" {  
			http.Error(w, "Unauthorized", http.StatusUnauthorized)  
			return  
		}  
  
		next.ServeHTTP(w, r)  
	})  
}

---

# 🔗 Цепочка Middleware

Можно объединять:

handler := LoggingMiddleware(  
	AuthMiddleware(  
		http.HandlerFunc(HelloHandler),  
	),  
)

Порядок важен.

---

# 🧵 Middleware и Context

Middleware часто добавляет данные в [[Context]]:

type key string  
  
func UserMiddleware(next http.Handler) http.Handler {  
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {  
		ctx := context.WithValue(r.Context(), key("userID"), 42)  
		next.ServeHTTP(w, r.WithContext(ctx))  
	})  
}

В handler:

func HelloHandler(w http.ResponseWriter, r *http.Request) {  
	userID := r.Context().Value(key("userID"))  
	fmt.Println("User:", userID)  
}

---

# 🚀 Middleware в gRPC

В gRPC middleware называется **Interceptor**.

Пример unary interceptor:

func LoggingInterceptor(  
	ctx context.Context,  
	req interface{},  
	info *grpc.UnaryServerInfo,  
	handler grpc.UnaryHandler,  
) (interface{}, error) {  
  
	log.Println("Method:", info.FullMethod)  
  
	resp, err := handler(ctx, req)  
  
	log.Println("Finished")  
  
	return resp, err  
}

---

# 🧠 Для чего middleware в реальных системах

В production middleware часто отвечает за:

- 🔍 Логирование
    
- 📊 Метрики
    
- 🔐 Аутентификацию
    
- 🛑 Rate limiting
    
- 🧯 Recovery от panic
    
- 📈 Трейсинг
    

---

# 📊 Метрики и мониторинг

Middleware часто используется вместе с:

- [[Prometheus]]
    
- [[Grafana]]
    
- [[OpenTelemetry]]
    
- [[Jaeger]]
    
- [[ELK Stack]]
    

Пример использования Prometheus middleware:

// Обычно через готовые библиотеки

---

# 🔐 Middleware и безопасность

Связано с:

- [[JWT]]
    
- [[OAuth]]
    
- [[TLS]]
    
- [[HTTPS]]
    

---

# ⚠ Частые ошибки

1️⃣ Забыли вызвать `next.ServeHTTP`

2️⃣ Нарушили порядок middleware

3️⃣ Передали данные через context без типизации ключей

4️⃣ Сделали слишком тяжёлую логику в middleware

---

# 📌 Главное понимание

Middleware — это:

> Механизм расширения обработки запроса без изменения основного handler.

Это фундамент архитектуры веб-приложений.

---

# 🔗 Связанные темы

- [[HTTP]]
    
- [[Context]]
    
- [[REST API]]
    
- [[gRPC]]
    
- [[JWT]]
    
- [[Prometheus]]
    
- [[Grafana]]
    
- [[OpenTelemetry]]
    
- [[Rate Limiting]]
    
- [[Logging]]
    
- [[Microservices]]
    

---

# 💎 Инсайт для тебя

Middleware — это точка:

- контроля
    
- наблюдения
    
- безопасности
    
- масштабирования
    

Если ты понимаешь middleware — ты понимаешь архитектуру backend.