# Pydantic & Annotated

<p class="note-meta">
  <span class="tag">Completed</span>
  <span>Updated 2026-08-06</span>
  <a href="https://docs.pydantic.dev/latest/concepts/models/">Source: Pydantic docs</a>
</p>

> Pydantic v2 · Python 3.10+ · engine đứng sau FastAPI

---

## 1. Pydantic là gì

Python có type hint nhưng **runtime không kiểm tra gì cả**:

```python
def f(x: int): return x + 1
f("5")   # TypeError lúc chạy, không ai chặn trước
```

Pydantic biến type hint thành **validation thật lúc runtime**: đọc annotation → kiểm tra → ép kiểu → báo lỗi có cấu trúc.

| Việc | Ví dụ |
|---|---|
| **Validate** | `age: int` nhận `"abc"` → lỗi rõ ràng |
| **Coerce** (ép kiểu) | `"5"` → `5`, `"2024-01-15"` → `datetime` |
| **Serialize** | model ↔ `dict` / JSON |

Đây chính là engine sau lưng FastAPI: request body, query param, response model đều là Pydantic.

---

## 2. BaseModel — nền tảng

```python
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str
    is_active: bool = True        # có default → optional
    nickname: str | None = None   # nullable VÀ optional

u = User(id="123", name="Đông")   # "123" được ép thành int 123
print(u.id, type(u.id))           # 123 <class 'int'>
```

Validate xảy ra ngay lúc `__init__` — **object đã tạo được thì chắc chắn hợp lệ**.

### Lỗi validation

```python
from pydantic import ValidationError

try:
    User(id="abc", name=123)
except ValidationError as e:
    for err in e.errors():
        print(err["type"], err["loc"])
    # int_parsing  ('id',)
    # string_type  ('name',)
```

`e.errors()` trả list dict có `loc` / `msg` / `type` / `input` — FastAPI dùng đúng cái này để sinh response 422.

> **Ép kiểu là bất đối xứng.** `"123"` → `123` được, nhưng `123` → `str` thì **không**. Pydantic chỉ ép khi phép chuyển không mất thông tin và không mơ hồ.

### Bẫy kinh điển: required vs optional

```python
class M(BaseModel):
    a: int                # BẮT BUỘC
    b: int | None         # BẮT BUỘC, nhưng cho phép None
    c: int | None = None  # KHÔNG bắt buộc
```

`| None` chỉ nói về **kiểu**, không nói về **bắt buộc hay không**. Muốn optional phải có **default**.

---

## 3. Coercion: lax vs strict

Mặc định Pydantic ở chế độ **lax** — ép kiểu khi "an toàn":

```python
class P(BaseModel):
    n: int
    f: float
    b: bool

P(n="42", f="3.14", b="yes")   # n=42, f=3.14, b=True
```

`bool` chấp nhận rất rộng: `"yes"/"no"`, `"true"/"false"`, `"on"/"off"`, `"1"/"0"`, `1/0`.

Muốn chặt chẽ:

```python
from pydantic import ConfigDict, Field
from typing import Annotated

class Strict(BaseModel):
    model_config = ConfigDict(strict=True)   # toàn model
    n: int

class Mixed(BaseModel):
    n: Annotated[int, Field(strict=True)]    # chỉ 1 field
    m: int                                   # vẫn lax
```

---

## 4. `Field()` — ràng buộc & metadata

```python
from pydantic import BaseModel, Field

class Product(BaseModel):
    name: str = Field(min_length=1, max_length=100)
    price: float = Field(gt=0)
    qty: int = Field(default=0, ge=0)
    sku: str = Field(pattern=r"^[A-Z]{3}-\d{4}$")
    tags: list[str] = Field(default_factory=list)
```

| Nhóm | Tham số |
|---|---|
| Số | `gt` `ge` `lt` `le` `multiple_of` |
| Chuỗi / list | `min_length` `max_length` `pattern` |
| Default | `default` `default_factory` |
| Docs | `description` `title` `examples` |
| Tên | `alias` `validation_alias` `serialization_alias` |
| Khác | `frozen` `exclude` `strict` `repr` |

**`default_factory`** — mỗi instance một object mới, ý đồ rõ ràng:

```python
tags: list[str] = Field(default_factory=list)
created: datetime = Field(default_factory=datetime.now)
```

**`alias`** — khi JSON có tên khác Python:

```python
class Cfg(BaseModel):
    model_config = ConfigDict(populate_by_name=True)
    db_url: str = Field(alias="databaseUrl")

Cfg(databaseUrl="postgres://...")   # theo alias
Cfg(db_url="postgres://...")        # theo tên field (nhờ populate_by_name)
```

---

## 5. `Annotated` — gắn metadata vào kiểu

`Annotated` đến từ `typing` (Python 3.9+), **không phải của Pydantic**.

```python
Annotated[<Kiểu thật>, <metadata 1>, <metadata 2>, ...]
```

Quy tắc: **phần tử đầu là kiểu thật**, mọi thứ sau là metadata mà Python **bỏ qua hoàn toàn lúc chạy** — chỉ thư viện nào chủ động đọc mới thấy.

```python
from typing import Annotated, get_type_hints

def f(x: Annotated[int, "ghi chú", 42]) -> None: ...

get_type_hints(f)                       # {'x': <class 'int'>, ...}
get_type_hints(f, include_extras=True)  # Annotated[int, 'ghi chú', 42]
```

Với type checker, `Annotated[int, ...]` **chính là** `int` — không mất gì về kiểu, chỉ đính thêm thông tin.

### Vì sao Pydantic/FastAPI cần nó

Trước `Annotated`, ràng buộc phải nhét vào **giá trị mặc định**:

```python
name: str = Field(min_length=1)   # kiểu: str, default: một object Field ???
```

Lộn xộn, và nó **chiếm mất chỗ default**. Với `Annotated`, ràng buộc về đúng chỗ của nó — trong phần kiểu:

```python
name: Annotated[str, Field(min_length=1)]           # bắt buộc
name: Annotated[str, Field(min_length=1)] = "n/a"   # có default thật
```

---

## 6. `Field()` ở default vs trong `Annotated`

Hai cách **tương đương** trong Pydantic:

```python
class A(BaseModel):
    x: int = Field(gt=0)             # style cũ

class B(BaseModel):
    x: Annotated[int, Field(gt=0)]   # style mới ✅
```

Nhưng `Annotated` thắng ở 3 điểm:

**a) Default trở lại đúng chỗ**

```python
x: Annotated[int, Field(gt=0)] = 10    # rõ ràng: default là 10
```

**b) Tái sử dụng được — lợi ích lớn nhất**

```python
PositiveInt = Annotated[int, Field(gt=0)]
Title       = Annotated[str, Field(min_length=1, max_length=120)]
Percent     = Annotated[float, Field(ge=0, le=100)]

class Task(BaseModel):
    id: PositiveInt
    title: Title
    progress: Percent = 0.0

class Project(BaseModel):
    id: PositiveInt        # dùng lại, không copy-paste ràng buộc
    title: Title
```

Style cũ không làm được — `x: int = Field(gt=0)` không tách ra thành biến dùng chung được.

**c) Dùng được ở nơi không có default**

```python
scores: list[Annotated[int, Field(ge=0, le=10)]]   # ràng buộc từng phần tử
```

> Ghi nhớ: `Annotated` mô tả **kiểu là gì**, dấu `=` mô tả **giá trị mặc định**. Đừng trộn hai việc đó.

---

## 7. Nested models

```python
from enum import Enum

class Priority(str, Enum):
    LOW = "low"
    HIGH = "high"

class Tag(BaseModel):
    name: str
    color: str = "#888"

class Task(BaseModel):
    title: str
    priority: Priority = Priority.LOW
    tags: list[Tag] = Field(default_factory=list)
    meta: dict[str, str] = Field(default_factory=dict)

t = Task.model_validate({
    "title": "Học Pydantic",
    "priority": "high",
    "tags": [{"name": "study"}, {"name": "python", "color": "#3776ab"}],
})

t.priority is Priority.HIGH   # True
t.tags[0].color               # "#888" — default áp dụng cho nested
```

Pydantic đệ quy xuống mọi tầng. `loc` trong lỗi cũng lồng theo: `('tags', 1, 'name')`.

---

## 8. Validators

### `field_validator` — kiểm 1 field

```python
from pydantic import field_validator

class User(BaseModel):
    username: str

    @field_validator("username")
    @classmethod
    def no_space(cls, v: str) -> str:
        if " " in v:
            raise ValueError("username không được có khoảng trắng")
        return v.lower()          # LUÔN phải return giá trị
```

- `mode="after"` (mặc định): chạy **sau** khi ép kiểu → `v` đã đúng kiểu.
- `mode="before"`: chạy **trước** → `v` là dữ liệu thô, dùng để chuẩn hoá input lạ.

```python
    @field_validator("tags", mode="before")
    @classmethod
    def split_str(cls, v):
        return v.split(",") if isinstance(v, str) else v
```

### `model_validator` — kiểm nhiều field cùng lúc

```python
from pydantic import model_validator
from typing_extensions import Self

class Range(BaseModel):
    start: int
    end: int

    @model_validator(mode="after")
    def check_order(self) -> Self:
        if self.start >= self.end:
            raise ValueError("start phải nhỏ hơn end")
        return self
```

### Validator trong `Annotated` — tái sử dụng được

```python
from pydantic import AfterValidator, BeforeValidator

def must_be_even(v: int) -> int:
    if v % 2: raise ValueError("phải là số chẵn")
    return v

EvenInt = Annotated[int, AfterValidator(must_be_even)]
Slug    = Annotated[str, BeforeValidator(lambda v: str(v).strip().lower())]

class M(BaseModel):
    n: EvenInt
    slug: Slug
```

Gọn hơn decorator khi rule dùng lại ở nhiều model.

---

## 9. Serialization

```python
u.model_dump()                     # -> dict
u.model_dump_json()                # -> chuỗi JSON
u.model_dump(exclude={"password"})
u.model_dump(exclude_none=True)
u.model_dump(exclude_unset=True)   # ⭐ chỉ field người dùng thực sự gửi
u.model_dump(by_alias=True)
```

### `exclude_unset` — chìa khoá cho PATCH

```python
class TaskUpdate(BaseModel):
    title: str | None = None
    done: bool | None = None

patch = TaskUpdate(done=True)
patch.model_dump()                    # {'title': None, 'done': True}  ❌ xoá mất title
patch.model_dump(exclude_unset=True)  # {'done': True}                 ✅
```

Không có `exclude_unset`, mọi PATCH sẽ vô tình ghi `None` đè lên field người dùng không đụng tới.

### Chiều ngược lại

```python
User.model_validate({"id": 1, "name": "A"})     # từ dict
User.model_validate_json('{"id":1,"name":"A"}') # từ chuỗi JSON
User.model_json_schema()                        # ra JSON Schema (nguồn của /docs)
```

---

## 10. `model_config`

```python
from pydantic import ConfigDict

class M(BaseModel):
    model_config = ConfigDict(
        strict=False,            # ép kiểu hay không
        frozen=True,             # immutable, hashable
        extra="forbid",          # "ignore" (mặc định) | "allow" | "forbid"
        populate_by_name=True,   # cho phép dùng tên field lẫn alias
        from_attributes=True,    # đọc từ ORM object (SQLAlchemy...)
        str_strip_whitespace=True,
    )
```

`extra="forbid"` rất đáng bật cho request body: client gõ sai tên field sẽ bị báo lỗi thay vì bị âm thầm bỏ qua.

---

## 11. `Annotated` trong FastAPI

FastAPI đọc metadata trong `Annotated` để biết tham số đến từ đâu:

```python
from typing import Annotated
from fastapi import FastAPI, Query, Path, Depends

@app.get("/tasks/{task_id}")
def read_task(
    task_id: Annotated[int, Path(ge=1)],
    q: Annotated[str | None, Query(max_length=50)] = None,
    limit: Annotated[int, Query(ge=1, le=100)] = 20,
    session: Annotated[Session, Depends(get_session)] = ...,
):
    ...
```

**Quy tắc vàng:** với `Annotated`, default đặt ở **sau dấu `=`**, không đặt trong `Query(default=...)`.

```python
q: Annotated[str, Query()] = "abc"      # ✅
q: Annotated[str, Query(default="abc")] # ❌ FastAPI báo lỗi
q: str = Query(default="abc")           # ⚠️ style cũ, vẫn chạy
```

Lợi ích thực tế lớn nhất — **dependency alias**:

```python
SessionDep  = Annotated[Session, Depends(get_session)]
CurrentUser = Annotated[User, Depends(get_current_user)]

@app.get("/me")
def me(user: CurrentUser): return user

@app.get("/tasks")
def list_tasks(session: SessionDep, user: CurrentUser): ...
```

Viết `Depends(...)` một lần, dùng ở trăm route.

---

## 12. Pydantic v1 → v2 (tra nhanh)

| v1 | v2 |
|---|---|
| `.dict()` | `.model_dump()` |
| `.json()` | `.model_dump_json()` |
| `.copy()` | `.model_copy()` |
| `.parse_obj()` | `.model_validate()` |
| `.parse_raw()` | `.model_validate_json()` |
| `.schema()` | `.model_json_schema()` |
| `@validator` | `@field_validator` |
| `@root_validator` | `@model_validator` |
| `class Config:` | `model_config = ConfigDict(...)` |
| `orm_mode` | `from_attributes` |
| `allow_population_by_field_name` | `populate_by_name` |
| `regex=` | `pattern=` |
| `min_items` / `max_items` | `min_length` / `max_length` |
| `Optional[X]` = optional | phải có default mới optional |

---

## 13. Bẫy hay gặp

1. **`X | None` không làm field thành optional.** Phải có `= None`.
2. **Validator quên `return`** → field thành `None`. Luôn trả về giá trị.
3. **`@field_validator` thiếu `@classmethod`** → hành vi lạ. Thứ tự: `@field_validator` trên, `@classmethod` dưới.
4. **Quên `exclude_unset` khi PATCH** → ghi `None` đè dữ liệu cũ.
5. **Trả thẳng model DB ra response** → lộ `hashed_password`. Luôn khai báo `response_model` riêng.
6. **`extra` mặc định là `"ignore"`** → client gõ sai tên field mà không hề biết. Cân nhắc `"forbid"`.
7. **Trong FastAPI, không đặt `default=` bên trong `Annotated`.** Pydantic thuần thì dễ tính, nhưng FastAPI chặn hẳn:
   ```python
   q: Annotated[str, Query(default="abc")]   # AssertionError:
   # `Query` default value cannot be set in `Annotated`. Set the default value with `=` instead.
   ```
8. **`bool` ép rất rộng**: `"no"`, `"off"`, `"0"` đều thành `False`. Dùng `strict=True` nếu cần chính xác.

---

## 14. Cheat sheet

```python
from typing import Annotated
from pydantic import BaseModel, Field, ConfigDict

# Type alias tái sử dụng
PositiveInt = Annotated[int, Field(gt=0)]
Title       = Annotated[str, Field(min_length=1, max_length=120)]

class TaskBase(BaseModel):
    model_config = ConfigDict(extra="forbid", str_strip_whitespace=True)
    title: Title
    priority: Priority = Priority.LOW

class TaskCreate(TaskBase):          # input
    pass

class TaskUpdate(BaseModel):         # PATCH: mọi field optional
    title: Title | None = None
    priority: Priority | None = None

class TaskPublic(TaskBase):          # output — quyết định cái gì lộ ra ngoài
    id: PositiveInt
    created_at: datetime
```

Bốn model cho một khái niệm — nghe thừa, nhưng đây chính là ranh giới giữa API đồ chơi và API production.

---

## Đọc thêm

- [Pydantic — Models](https://docs.pydantic.dev/latest/concepts/models/)
- [Pydantic — Fields](https://docs.pydantic.dev/latest/concepts/fields/)
- [Pydantic — Validators](https://docs.pydantic.dev/latest/concepts/validators/)
- [FastAPI — Query Params & String Validations](https://fastapi.tiangolo.com/tutorial/query-params-str-validations/)
- [FastAPI — Extra Models](https://fastapi.tiangolo.com/tutorial/extra-models/)
- [PEP 593 — Annotated](https://peps.python.org/pep-0593/)
