# Romusha DSL

Romusha adalah DSL (Domain Specific Language) berbasis JSON untuk mendeklarasikan logika, ekspresi, dan workflow secara modular, agnostik, dan lintas platform.

## Filosofi

Romusha adalah "pekerja" logika. Ia tidak menulis perintah, tapi mengeksekusi deklarasi — netral, modular, dan agnostic.

## Tujuan

- Menyediakan struktur deklaratif untuk menyusun logika dan proses
- Modular dan reusable, dapat dipecah menjadi modul dan task
- Dirancang untuk dieksekusi lintas bahasa

## Jenis File

| Type    | Deskripsi                                                                 |
|---------|--------------------------------------------------------------------------|
| script  | File utama berisi logika penuh dan eksekusi                              |
| module  | Unit logika modular yang dapat diimpor                                   |
| task    | Tugas terstruktur atau unit kerja mandiri                                |

## Struktur Dasar File
```json
{
  "type": "script",
  "logic": { }
}
```

## Node Ekspresi

Menjalankan fungsi yang ditentukan dalam `fn`.

```json
{
  "call": {
    "fn": "print",
    "args": [ { "value": "Hello" } ]
  }
}
```

Mendukung `injectAs` untuk menyimpan hasil ke `context`:

```json
{
  "call": {
    "fn": "getUserData",
    "args": [ { "var": "userId" } ],
    "injectAs": "user"
  }
}
```

### `steps`

Menjalankan array langkah secara berurutan.

```json
{
  "steps": [
    { "call": { "fn": "print", "args": ["Langkah 1"] } },
    { "call": { "fn": "print", "args": ["Langkah 2"] } }
  ]
}
```

### `if / elseIf / else`

Percabangan logika.

```json
{
  "if": { "call": { "fn": "equals", "args": [ { "var": "x" }, { "value": 1 } ] } },
  "then": { "call": { "fn": "print", "args": ["Sama"] } },
  "elseIf": [
    {
      "if": { "call": { "fn": "equals", "args": [ { "var": "x" }, { "value": 2 } ] } },
      "then": { "call": { "fn": "print", "args": ["Dua"] } }
    }
  ],
  "else": { "call": { "fn": "print", "args": ["Lainnya"] } }
}
```

### `each`

Melakukan iterasi terhadap array dengan binding item dan index.
```json
{
  "call": {
    "fn": "each",
    "args": [
      { "var": "items" },
      {
        "itemAs": "item",
        "indexAs": "i",
        "do": {
          "call": { "fn": "print", "args": [ { "var": "item" } ] }
        }
      }
    ]
  }
}
```

### `repeat`

Menjalankan ekspresi berulang kali, baik inline maupun sebagai node.

Inline dalam call:
```json
{
  "call": {
    "fn": "fetchData",
    "args": [ { "var": "id" } ],
    "repeat": 3
  }
}
```

Sebagai node eksplisit:

```json
{
  "repeat": {
    "times": 3,
    "do": {
      "call": { "fn": "pingServer" }
    }
  }
}
```

Dengan delayMs dan injectAs:

```json
{
  "repeat": {
    "times": 3,
    "delayMs": 1000,
    "injectAs": "resultSet",
    "do": {
      "call": { "fn": "fetchMetrics" }
    }
  }
}
```

### `and, or, not`
Evaluasi logika boolean.

```json
{ "and": ["condition1", "condition2"] }
{ "or": ["condition1", "condition2"] }
{ "not": "condition" }
```

## Argumen

### `args` (positional)
```json
{
  "call": {
    "fn": "sum",
    "args": [ { "var": "a" }, { "value": 10 } ]
  }
}
```
### `namedArgs` (berbasis key)
```json
{
  "call": {
    "fn": "sendEmail",
    "namedArgs": {
      "to": { "var": "user.email" },
      "subject": { "value": "Hello" },
      "body": { "value": "Selamat datang" }
    }
  }
}
```

## Konteks Evaluasi

Gunakan var untuk mengambil nilai dari context runtime.
```json
{ "var": "user.name" }
```

```json
{
  "call": {
    "fn": "increment",
    "args": [ { "var": "count" } ],
    "injectAs": "count"
  }
}
```

Hasil dari `call` dapat di-`injectAs` ke `context` baru untuk digunakan langkah berikutnya.

Ini membantu user memahami bahwa `context` bisa berubah secara `stateful` lewat `injectAs`.

## Plugin
Fungsi `call(fn)` akan di-`resolve` dari sistem plugin, mendukung:

- Built-in plugins (standar)
- Plugin eksternal/user-defined

## Rencana Selanjutnya
- Menyusun JSON Schema resmi untuk semua struktur DSL
- Evaluator awal (JavaScript, PHP)
- CLI runner & playground web
- Standar plugin library: print, sum, gte, each, dll

### Fitur Pelengkap

Berikut adalah fitur tambahan opsional namun powerful yang memperluas ekspresivitas Romusha DSL:

### `let` Bind Variable

Digunakan untuk mendefinisikan variabel lokal dari hasil ekspresi.
```json
{
  "let": {
    "userAge": { "call": { "fn": "getAge", "args": [ { "var": "user" } ] } },
    "isAdult": { "call": { "fn": "gte", "args": [ { "var": "userAge" }, { "value": 18 } ] } }
  }
}
```

### `while` Loop Dinamis

Menjalankan aksi selama kondisi bernilai true.

```json
{
  "while": {
    "condition": { "call": { "fn": "lte", "args": [ { "var": "count" }, { "value": 5 } ] } },
    "do": {
      "steps": [
        { "call": { "fn": "print", "args": [ { "var": "count" } ] } },
        { "call": { "fn": "increment", "args": [ { "var": "count" } ], "injectAs": "count" } }
      ]
    }
  }
}
```

### `try` / `catch` / `finally`
Menangani error saat evaluasi logika.
```json
{
  "try": {
    "do": { "call": { "fn": "fetchData" } },
    "catch": { "call": { "fn": "logError", "args": ["Gagal fetch"] } },
    "finally": { "call": { "fn": "log", "args": ["Selesai"] } }
  }
}
```

### `timeoutMs` dan `onTimeout`
```json
{
  "call": {
    "fn": "longProcess",
    "timeoutMs": 5000,
    "onTimeout": { "call": { "fn": "log", "args": ["Waktu habis"] } }
  }
}
```
### `import` dan `use`
Memungkinkan reuse logika antar file.
```
{
  "import": {
    "name": "getUser",
    "from": "./modules/getUser.json"
  }
}

{
  "use": "getUser"
}
```

### `map` / `filter` / `transform`

Manipulasi array terstruktur.
```json
{
  "map": {
    "from": { "var": "users" },
    "itemAs": "user",
    "do": {
      "call": { "fn": "getUserId", "args": [ { "var": "user" } ] }
    },
    "injectAs": "userIds"
  }
}
```
### `expr` Evaluasi Ekspresi Tunggal

Ekspresi satu langkah yang menghasilkan nilai.
```json
{
  "expr": {
    "call": { "fn": "sum", "args": [1, 2] }
  }
}
```

### `switch` / `case`
```json
{
  "switch": {
    "on": { "var": "status" },
    "cases": [
      { "match": "pending", "do": { "call": { "fn": "log", "args": ["Menunggu"] } } },
      { "match": "done", "do": { "call": { "fn": "log", "args": ["Selesai"] } } }
    ],
    "default": { "call": { "fn": "log", "args": ["Tidak dikenal"] } }
  }
}
```

### `alias` Nama Pendek untuk Fungsi
```json
{
  "alias": {
    "show": { "fn": "print" }
  }
}

{
  "call": { "alias": "show", "args": ["Hello"] }
}
```

### `meta` Informasi Tambahan
Dapat digunakan untuk debugging, label internal, atau anotasi.
```json
{
  "call": {
    "fn": "log",
    "args": ["Starting"],
    "meta": {
      "label": "init-log",
      "debug": true
    }
  }
}
```
### `noop` Ekspresi Kosong
Tidak melakukan apa pun.
```json
{ "noop": true }
```

## Next Plan DSL
Romusha dirancang sebagai DSL logika deklaratif yang modular dan extensible. Untuk menjadikannya lebih ekspresif dan setara dengan bahasa scripting modern, berikut adalah roadmap fitur lanjutan:

### Pipeline Expression (`pipe`)

Transformasi berantai seperti Unix pipe:
```json
{
  "pipe": [
    { "call": { "fn": "fetchUser" } },
    { "call": { "fn": "extractName" } },
    { "call": { "fn": "print" } }
  ]
}
```

### Penjelasan Perbedaan `pipe` vs `steps`

| Fitur          | Input Otomatis | Mutable Context | Cocok untuk          |
|----------------|----------------|------------------|----------------------|
| **pipe**       | ✔ Ya (`$`)     | ✕ Tidak          | Transformasi data    |
| **steps**      | ✕ Manual (`var`)| ✔ Ya (`injectAs`)| Workflow kompleks    |


### Inline Assignment (`set`, `as`)

Pendekatan penugasan yang lebih ringkas:
```json
{
  "call": {
    "fn": "getTotal",
    "args": [ { "var": "items" } ],
    "as": "total"
  }
}

{
  "set": {
    "total": { "call": { "fn": "sum", "args": ["..."] } }
  }
}
```

### Define Function (`define`)
```json
{
  "define": {
    "fn": "greetUser",
    "params": ["name"],
    "body": {
      "call": { "fn": "print", "args": [ { "var": "name" } ] }
    }
  }
}
```

### Documentation & Comment
Menambahkan dokumentasi pada setiap node:
```json
{
  "doc": "Langkah ini mencetak nama user",
  "call": { "fn": "print", "args": [ { "var": "user.name" } ] }
}
```
### Structural Pattern Matching (`match`)
Mendukung logika berbasis struktur data:
```json
{
  "match": {
    "value": { "var": "event" },
    "cases": [
      {
        "pattern": { "type": "login" },
        "do": { "call": { "fn": "trackLogin" } }
      },
      {
        "pattern": { "type": "logout" },
        "do": { "call": { "fn": "trackLogout" } }
      }
    ]
  }
}
```

### Remote Module
Mengimpor logika dari URL eksternal:
```json
{
  "import": {
    "name": "getWeather",
    "from": "https://example.com/plugin/weather.json"
  }
}
```

### Scoped Context (`with`)
Menjalankan blok dengan context terisolasi:
```json
{
  "with": {
    "context": {
      "user": { "call": { "fn": "getUser" } }
    },
    "do": {
      "call": { "fn": "print", "args": [ { "var": "user.name" } ] }
    }
  }
}
```

### Feature Flag Handling

Memungkinkan conditional behavior:
```json
{
  "ifFeature": "beta_mode",
  "then": { "call": { "fn": "print", "args": ["Beta aktif"] } }
}
```
### Plugin Ekosistem yang Direncanakan

- File I/O: readFile, writeFile
- HTTP: http.get, http.post
- Date/Time: now, addDays, formatDate
- Env: getEnv
- Crypto: hash, compare
