# 🌐 HTTP в Go

## Что такое HTTP

HTTP — это прикладной протокол по модели request/response.

Работает поверх:

- [[TCP]]
    
- используется в [[REST API]]
    
- взаимодействует с [[JSON]]
    
- маршрутизируется через [[Router]]
    

---

# 🖥 Поднятие базового HTTP-сервера

## Самый простой сервер

package main  
  
import (  
	"fmt"  
	"net/http"  
)  
  
func main() {  
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {  
		fmt.Fprintln(w, "Hello World")  
	})  
  
	http.ListenAndServe(":8080", nil)  
}

Запуск:

go run main.go

Открываешь:

http://localhost:8080

---

# 🧱 Правильный способ с Server

Лучше явно создавать сервер:

package main  
  
import (  
	"log"  
	"net/http"  
	"time"  
)  
  
func main() {  
	mux := http.NewServeMux()  
	mux.HandleFunc("/", homeHandler)  
  
	server := &http.Server{  
		Addr:         ":8080",  
		Handler:      mux,  
		ReadTimeout:  5 * time.Second,  
		WriteTimeout: 10 * time.Second,  
	}  
  
	log.Println("Server started on :8080")  
	log.Fatal(server.ListenAndServe())  
}  
  
func homeHandler(w http.ResponseWriter, r *http.Request) {  
	w.Write([]byte("Home Page"))  
}

Почему так лучше:

- можно управлять таймаутами
    
- лучше контроль
    
- production-ready
    

---

# 📥 Работа с HTTP-запросом

## Чтение параметров query

GET /search?q=golang

func searchHandler(w http.ResponseWriter, r *http.Request) {  
	query := r.URL.Query().Get("q")  
	w.Write([]byte("Search: " + query))  
}

---

## Чтение JSON из body

type User struct {  
	Name string `json:"name"`  
}  
  
func createUser(w http.ResponseWriter, r *http.Request) {  
	var user User  
  
	err := json.NewDecoder(r.Body).Decode(&user)  
	if err != nil {  
		http.Error(w, "Invalid JSON", http.StatusBadRequest)  
		return  
	}  
  
	w.Write([]byte("Created user: " + user.Name))  
}

---

# 📤 Отправка JSON-ответа

func getUser(w http.ResponseWriter, r *http.Request) {  
	user := User{Name: "Nazar"}  
  
	w.Header().Set("Content-Type", "application/json")  
	json.NewEncoder(w).Encode(user)  
}

---

# 🔢 Работа со статус-кодами

http.Error(w, "Not found", http.StatusNotFound)

Или вручную:

w.WriteHeader(http.StatusCreated)  
w.Write([]byte("Created"))

Важно:

> WriteHeader нужно вызвать до Write()

---

# 🧵 Работа с Context

Каждый запрос содержит контекст:

func handler(w http.ResponseWriter, r *http.Request) {  
	ctx := r.Context()  
  
	select {  
	case <-time.After(5 * time.Second):  
		w.Write([]byte("Done"))  
	case <-ctx.Done():  
		return  
	}  
}

Если клиент закрыл соединение → контекст отменится.

---

# 🌐 HTTP Client в Go

## Простой запрос

resp, err := http.Get("https://jsonplaceholder.typicode.com/posts/1")  
if err != nil {  
	log.Fatal(err)  
}  
defer resp.Body.Close()  
  
body, _ := io.ReadAll(resp.Body)  
fmt.Println(string(body))

---

## POST-запрос с JSON

user := User{Name: "Nazar"}  
data, _ := json.Marshal(user)  
  
resp, err := http.Post(  
	"http://localhost:8080/users",  
	"application/json",  
	bytes.NewBuffer(data),  
)

---

# 🧠 Правильный HTTP Client

Лучше использовать кастомный клиент:

client := &http.Client{  
	Timeout: 5 * time.Second,  
}  
  
req, _ := http.NewRequest("GET", "https://api.example.com", nil)  
req.Header.Set("Authorization", "Bearer token")  
  
resp, err := client.Do(req)

---

# 🔐 HTTPS сервер

server.ListenAndServeTLS("cert.pem", "key.pem")

Связано с:

- [[TLS]]
    
- [[Certificates]]
    

---

# 🛑 Graceful Shutdown

Очень важно для production:

ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt)  
defer stop()  
  
go func() {  
	server.ListenAndServe()  
}()  
  
<-ctx.Done()  
server.Shutdown(context.Background())

Позволяет:

- завершить активные запросы
    
- корректно закрыть соединения
    

---

# 🧩 Структура реального HTTP-сервиса

main.go  
router.go  
handler.go  
service.go  
repository.go

Связано с:

- [[Clean Architecture]]
    
- [[Dependency Injection]]
    
- [[Middleware]]
    

---

# 📌 Что происходит под капотом

Когда приходит HTTP-запрос:

1. [[TCP]] принимает соединение
    
2. HTTP парсит заголовки
    
3. Router находит handler
    
4. Middleware оборачивает handler
    
5. Handler выполняется
    
6. Ответ сериализуется ([[JSON]])
    
7. TCP отправляет обратно
    

---

# 🔗 Связанные темы

- [[Router]]
    
- [[Middleware]]
    
- [[Context]]
    
- [[JSON]]
    
- [[REST API]]
    
- [[TCP]]
    
- [[Socket]]
    
- [[TLS]]
    
- [[Status Code]]
    
- [[Chi Router]]
    
- [[gRPC]]
    

---

# 💎 Главное понимание

HTTP в Go — это:

> Обёртка над TCP, которая позволяет писать веб-сервисы через handler’ы.

Ты управляешь:

- маршрутизацией
    
- сериализацией
    
- контекстом
    
- middleware
    
- временем жизни соединений