Блаюлаюдала
# 🧩 Dependency Injection (DI)

Dependency Injection — это способ **передавать зависимости в объект извне**, а не создавать их внутри.

Проще:

> Объект не создаёт то, что ему нужно.  
> Ему это передают.

Связано с:

- [[Interfaces]]
    
- [[Clean Architecture]]
    
- [[Testing]]
    
- [[Router]]
    
- [[Middleware]]
    
- [[HTTP]]
    

---

# 🧠 В чём проблема без DI

Плохой пример:

type UserHandler struct{}  
  
func (h *UserHandler) GetUsers(w http.ResponseWriter, r *http.Request) {  
	db := NewPostgresDB() // ❌ создаём внутри  
	users := db.GetAll()  
	...  
}

Проблемы:

- нельзя протестировать
    
- нельзя заменить БД
    
- сильная связность
    
- нарушается [[Clean Architecture]]
    

---

# ✅ Как правильно (с DI)

type UserHandler struct {  
	repo UserRepository  
}  
  
func NewUserHandler(repo UserRepository) *UserHandler {  
	return &UserHandler{repo: repo}  
}

Теперь:

- handler не знает какая БД
    
- можно подставить mock
    
- код гибкий
    

---

# 🔗 Связь с интерфейсами

DI почти всегда работает через [[Interfaces]].

type UserRepository interface {  
	GetAll() []User  
}

Теперь можно сделать:

type PostgresRepo struct{}  
type MockRepo struct{}

И подставлять любую реализацию.

---

# 🚀 Пример DI в HTTP сервере

func main() {  
	repo := NewPostgresRepo()  
	handler := NewUserHandler(repo)  
  
	r := chi.NewRouter() // [[Chi Router]]  
  
	r.Get("/users", handler.GetUsers)  
  
	http.ListenAndServe(":8080", r)  
}

Здесь:

- main создаёт зависимости
    
- handler только использует их
    

Это правильная точка сборки приложения.

---

# 🧠 DI и Middleware

DI часто используется для:

- logger
    
- database
    
- config
    
- cache
    

Пример:

type App struct {  
	Logger *log.Logger  
	DB     *sql.DB  
}

И передаётся в handler:

func (a *App) UsersHandler(w http.ResponseWriter, r *http.Request) {  
	a.Logger.Println("request received")  
}

---

# 🔥 DI и Testing

Без DI тесты боль.

С DI:

func TestHandler(t *testing.T) {  
	mockRepo := &MockRepo{}  
	handler := NewUserHandler(mockRepo)  
  
	req := httptest.NewRequest("GET", "/users", nil)  
	rec := httptest.NewRecorder()  
  
	handler.GetUsers(rec, req)  
}

Связано с:

- [[Testing]]
    
- httptest
    

---

# 📦 Виды Dependency Injection

### 1️⃣ Constructor Injection (самый правильный)

Передаём через конструктор.

NewUserHandler(repo)

---

### 2️⃣ Field Injection (редко используется в Go)

Заполняем поля напрямую.

---

### 3️⃣ Interface-based DI

Через интерфейсы (самый гибкий способ в Go).

---

# 🧠 DI и Clean Architecture

В [[Clean Architecture]]:

- внешний слой зависит от внутреннего
    
- бизнес-логика не зависит от БД
    
- зависимости направлены внутрь
    

DI помогает это реализовать.

---

# 📊 Плюсы DI

✅ Легче тестировать  
✅ Слабая связность  
✅ Гибкость  
✅ Легче менять реализацию  
✅ Чистая архитектура

---

# ❌ Минусы

❌ Больше кода  
❌ Нужно понимать архитектуру  
❌ Сложнее для новичков

---

# 🧠 Важно понять

DI — это не библиотека.  
Это архитектурный подход.

В Go чаще всего:

- руками создаём зависимости в `main`
    
- передаём через конструкторы
    
- используем интерфейсы
    

---

# 🎯 Как это выглядит в реальном API

main  
 ↓  
create DB  
create Logger  
create Repo  
create Service  
create Handler  
 ↓  
Router  
 ↓  
Middleware  
 ↓  
Handler

Именно поэтому в твоём графе он связан с:

- [[API]]
    
- [[HTTP]]
    
- [[Testing]]
    
- [[Interfaces]]
    
- [[Clean Architecture]]
    

---

# 🔗 Связанные темы

- [[Interfaces]]
    
- [[Clean Architecture]]
    
- [[Testing]]
    
- [[Router]]
    
- [[Middleware]]
    
- [[HTTP]]
    
- [[Struct]]