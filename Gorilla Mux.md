# 🦍 Gorilla Mux (Go): практическое использование

## Что такое gorilla/mux

**gorilla/mux** — это расширенный [[Router]] для [[HTTP]].

Он:

- работает поверх стандартного [[HTTP]]
    
- поддерживает параметры пути
    
- умеет матчить методы, хосты, query
    
- поддерживает [[Middleware]]
    
- удобен для [[REST API]]
    

Главная мысль:

> mux — это гибкий роутер с чуть более “богатым” функционалом, чем стандартный ServeMux.

Связанные темы: [[Router]], [[HTTP]], [[Middleware]], [[REST API]]

---

# 🚀 Установка

go get github.com/gorilla/mux

Импорт:

import "github.com/gorilla/mux"

---

# 🏗 Базовый пример

package main  
  
import (  
	"net/http"  
  
	"github.com/gorilla/mux"  
)  
  
func main() {  
	r := mux.NewRouter()  
  
	r.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {  
		w.Write([]byte("hello"))  
	}).Methods(http.MethodGet)  
  
	http.ListenAndServe(":8080", r)  
}

Обрати внимание:

.Methods(http.MethodGet)

mux позволяет явно ограничивать [[HTTP Methods]].

---

# 📌 Регистрация маршрутов (практика)

В реальном API обычно делают так:

r := mux.NewRouter()  
  
r.HandleFunc("/api/v1/users", listUsers).Methods("GET")  
r.HandleFunc("/api/v1/users", createUser).Methods("POST")  
r.HandleFunc("/api/v1/users/{id}", getUser).Methods("GET")  
r.HandleFunc("/api/v1/users/{id}", deleteUser).Methods("DELETE")

Это читается как декларация API-контракта.

Связанные темы: [[Versioning]], [[Status Code]]

---

# 📦 Параметры пути

mux поддерживает параметры через `{}`:

r.HandleFunc("/users/{id}", getUser).Methods("GET")

В handler:

func getUser(w http.ResponseWriter, r *http.Request) {  
	vars := mux.Vars(r)  
	id := vars["id"]  
  
	w.Write([]byte("user id = " + id))  
}

Это намного удобнее, чем вручную разбирать URL.

Связанные темы: [[Router]]

---

# 🧭 Подроутеры (Subrouter)

Очень полезная фича для структурирования API.

api := r.PathPrefix("/api/v1").Subrouter()  
  
api.HandleFunc("/users", listUsers).Methods("GET")  
api.HandleFunc("/users", createUser).Methods("POST")

Теперь не нужно повторять `/api/v1` в каждом маршруте.

Это делает код чище и масштабируемым.

---

# 🔐 Middleware в gorilla/mux

Middleware подключаются через:

r.Use(loggingMiddleware)

Сигнатура такая же, как в [[HTTP]]:

func(next http.Handler) http.Handler

Пример:

func loggingMiddleware(next http.Handler) http.Handler {  
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {  
		log.Println("start:", r.Method, r.URL.Path)  
  
		next.ServeHTTP(w, r)  
  
		log.Println("end:", r.Method, r.URL.Path)  
	})  
}

---

# 🎯 Middleware только для части маршрутов

Через Subrouter:

api := r.PathPrefix("/api/v1").Subrouter()  
api.Use(authMiddleware)  
  
api.HandleFunc("/users", listUsers).Methods("GET")

Публичные маршруты можно оставить на основном роутере.

Это удобно для разделения:

- публичного API
    
- защищённого API
    

Связанные темы: [[JWT]], [[Authentication]], [[Authorization]]

---

# 🔎 Матчинг по Query, Host, Headers

mux умеет больше, чем просто путь.

### По query-параметру:

r.HandleFunc("/search", searchHandler).  
	Methods("GET").  
	Queries("q", "{query}")

### По заголовку:

r.HandleFunc("/api", handler).  
	Headers("Content-Type", "application/json")

### По хосту:

r.Host("admin.example.com").  
	HandleFunc("/dashboard", adminHandler)

Это полезно в сложных [[REST API]] или multi-tenant системах.

---

# 📤 Работа с JSON

mux не навязывает способ сериализации.

Обычно:

json.NewEncoder(w).Encode(data)

И:

json.NewDecoder(r.Body).Decode(&req)

Связанные темы: [[JSON]], [[Serialization]]

---

# ⚠ Частые ошибки при работе с mux

## ❌ Забыл `.Methods(...)`

Если не указать метод, маршрут будет принимать все методы.

Это может быть небезопасно.

---

## ❌ Двойной слэш

`/api//v1/users` ≠ `/api/v1/users`

mux не нормализует путь автоматически.

Связанные темы: [[Router]], [[HTTP]]

---

## ❌ Регистрация маршрутов после запуска сервера

Все `HandleFunc` должны быть вызваны ДО:

http.ListenAndServe(...)

---

# 🧪 Тестирование mux

mux — это `http.Handler`, поэтому тестирование стандартное:

req := httptest.NewRequest("GET", "/users/1", nil)  
rec := httptest.NewRecorder()  
  
r.ServeHTTP(rec, req)  
  
if rec.Code != http.StatusOK {  
	t.Fatalf("expected 200")  
}

Никакой магии.

Связанные темы: [[Testing]], [[httptest]]

---

# 🧠 Когда использовать gorilla/mux

Подходит если:

- нужен более гибкий матчинг
    
- хочется Subrouter
    
- нужен контроль по host/query/header
    
- проект уже использует mux
    

Если нужен более “тонкий” вариант — см. [[Chi Router]].

---

# 🆚 mux vs стандартный ServeMux

|Возможность|ServeMux|mux|
|---|---|---|
|Параметры пути|❌|✅|
|Методы|вручную|`.Methods()`|
|Subrouter|❌|✅|
|Middleware|вручную|`.Use()`|

---

# 🧭 Итоговая модель

gorilla/mux =

- маршрутизация
    
- параметры пути
    
- методы
    
- subrouter
    
- middleware
    
- гибкий матчинг
    

Он не:

- не бизнес-слой
    
- не ORM
    
- не DI-контейнер
    
- не full framework
    

Это просто расширенный [[Router]] поверх [[HTTP]].