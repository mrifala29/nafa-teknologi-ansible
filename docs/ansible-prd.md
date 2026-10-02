# Product Requirements Document (PRD): Standard Ansible Project Structure

## 1. Tujuan

Membuat standard struktur repository Ansible yang konsisten dan reusable untuk seluruh project.

Struktur harus:

* Mudah dipahami oleh engineer.
* Modular dan scalable.
* Memisahkan environment dari automation logic.
* Mendukung penggunaan Ansible Roles.
* Mendukung instalasi/configuration secara spesifik menggunakan `--tags`.

---

## 2. Standard Directory Structure

Setiap project yang menggunakan Ansible menggunakan struktur dasar berikut:

```text
ansible/
├── ansible.cfg
│
├── inventories/
│   ├── dev/
│   │   ├── hosts.ini
│   │   ├── group_vars/
│   │   │   └── all.yml
│   │   └── host_vars/
│   │       └── ...
│   │
│   ├── staging/
│   │   ├── hosts.ini
│   │   ├── group_vars/
│   │   │   └── all.yml
│   │   └── host_vars/
│   │       └── ...
│   │
│   └── prod/
│       ├── hosts.ini
│       ├── group_vars/
│       │   └── all.yml
│       └── host_vars/
│           └── ...
│
├── playbooks/
│   ├── setup_dev.yml
│   ├── setup_staging.yml
│   └── setup_prod.yml
│
└── roles/
    ├── <category>/
    │   └── <service>/
    │       ├── tasks/
    │       │   └── main.yml
    │       ├── defaults/
    │       │   └── main.yml
    │       ├── handlers/
    │       │   └── main.yml
    │       ├── templates/
    │       │   └── ...
    │       └── files/
    │           └── ...
    │
    └── ...
```

---

## 3. Folder Responsibilities

### `ansible.cfg`

Configuration utama Ansible untuk project.

---

### `inventories/`

Berisi inventory yang dipisahkan berdasarkan environment.

```text
inventories/
├── dev/
├── staging/
└── prod/
```

Setiap environment dapat memiliki:

* `hosts.ini` → daftar host dan grouping.
* `group_vars/` → variable berdasarkan group.
* `host_vars/` → variable khusus host.

---

### `playbooks/`

Berisi entrypoint untuk menjalankan automation berdasarkan environment.

Contoh:

```text
playbooks/
├── setup_dev.yml
├── setup_staging.yml
└── setup_prod.yml
```

Playbook bertugas memanggil roles, bukan menampung seluruh logic configuration.

---

### `roles/`

Berisi automation yang dibagi berdasarkan category dan service/function.

```text
roles/
└── <category>/
    └── <service>/
```

Contoh struktur **hanya sebagai ilustrasi**, bukan daftar role yang diwajibkan:

```text
roles/
├── base/
│   └── <service>/
├── database/
│   └── <service>/
├── monitoring/
│   └── <service>/
└── ...
```

Setiap role dapat memiliki:

| Folder       | Fungsi                         |
| ------------ | ------------------------------ |
| `tasks/`     | Task utama                     |
| `defaults/`  | Default variables              |
| `handlers/`  | Restart/reload service         |
| `templates/` | Jinja2 configuration templates |
| `files/`     | Static files                   |

Folder yang tidak dibutuhkan tidak perlu dibuat.

---

## 4. Naming Convention

Gunakan naming yang konsisten:

```text
setup_<environment>.yml
```

Untuk role:

```text
roles/<category>/<service>
```

Variable menggunakan `snake_case`:

```yaml
service_port: 8080
service_version: "1.0.0"
```

---

## 5. Tags

Role yang dapat dijalankan secara independen sebaiknya memiliki tag berdasarkan category dan service.

Contoh:

```yaml
tags:
  - database
  - <service>
```

Sehingga dapat dijalankan secara spesifik:

```bash
ansible-playbook \
  -i inventories/prod/hosts.ini \
  playbooks/setup_prod.yml \
  --tags "<service>"
```

Atau berdasarkan category:

```bash
ansible-playbook \
  -i inventories/prod/hosts.ini \
  playbooks/setup_prod.yml \
  --tags "database"
```

---

## 6. Prinsip

Standard ini merupakan **baseline structure**, bukan struktur yang kaku.

Project dapat menambah atau mengurangi:

* Environment
* Category
* Role
* Variable files
* Template
* File
* Playbook

sesuai kebutuhan.

Yang harus dipertahankan adalah pemisahan utama:

```text
Inventory
    ↓
Playbook
    ↓
Roles
    ↓
Tasks / Templates / Handlers
```

Tujuannya adalah agar struktur Ansible antar project tetap konsisten dan engineer dapat memahami repository baru tanpa harus mempelajari struktur yang berbeda-beda.
