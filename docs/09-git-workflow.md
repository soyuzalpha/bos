# Git Workflow Guide

## Tujuan

Menjaga branch `main` tetap stabil dan siap dijadikan source of truth project.

Seluruh eksperimen, trial-error, hasil generate AI, refactor besar, dan fitur baru dilakukan di branch lain terlebih dahulu sebelum masuk ke `main`.

---

# Branch Strategy

## Main Branch

Branch utama yang harus selalu stabil.

**Rules:**

- Tidak ngoding langsung di `main`
- Tidak melakukan experiment di `main`
- Hanya menerima hasil merge yang sudah diuji

Contoh:

```bash
git checkout main
git pull origin main
```

---

## Develop Branch

Branch kerja harian.

Semua development dilakukan di sini.

Contoh:

```bash
git checkout -b develop
git push -u origin develop
```

Workflow harian:

```bash
git checkout develop
git pull origin develop
```

---

## Feature Branch

Digunakan untuk fitur besar atau pekerjaan yang terpisah.

Format:

```text
feature/auth
feature/product
feature/customer
feature/transaction
feature/report
```

Membuat feature branch:

```bash
git checkout develop
git checkout -b feature/auth
```

Setelah selesai:

```bash
git add .
git commit -m "feat(auth): implement login"
```

Merge kembali ke develop:

```bash
git checkout develop
git merge feature/auth
```

Hapus branch:

```bash
git branch -d feature/auth
```

---

# Recommended Workflow

## Daily Development

Masuk ke branch develop:

```bash
git checkout develop
git pull
```

Kerjakan fitur.

Commit sesering mungkin.

Contoh:

```bash
git commit -m "feat(product): create product module"
git commit -m "feat(product): add category relation"
git commit -m "fix(product): resolve validation issue"
```

Jangan menunggu semua pekerjaan selesai baru commit.

---

# Merge To Main

Ketika satu milestone selesai:

Contoh:

- Authentication selesai
- Product selesai
- Transaction selesai
- Inventory selesai

Merge develop ke main:

```bash
git checkout main
git pull

git merge develop

git push origin main
```

---

# Emergency Recovery

Jika codingan rusak sebelum commit:

Buang seluruh perubahan:

```bash
git restore .
```

Atau:

```bash
git reset --hard HEAD
```

---

# Commit Convention

## Feature

```bash
feat(auth): implement login endpoint
```

## Fix

```bash
fix(product): resolve duplicate sku validation
```

## Refactor

```bash
refactor(order): simplify service structure
```

## Documentation

```bash
docs(api): update swagger examples
```

## Chore

```bash
chore(prisma): update migration files
```

---

# Project Structure

```text
bos/
├── docs/
│   ├── 00-index.md
│   ├── 01-business-domain-analysis.md
│   ├── ...
│   └── 09-git-workflow.md
│
├── backend/
│   ├── src/
│   ├── prisma/
│   └── ...
│
├── frontend/
│   ├── app/
│   ├── components/
│   └── ...
│
└── README.md
```

---

# Golden Rules

1. Jangan coding langsung di `main`
2. Commit kecil dan sering
3. Satu commit untuk satu tujuan
4. Jangan push code yang belum pernah dijalankan
5. Jangan merge code yang belum diuji
6. Jangan gunakan branch sebagai folder project
7. Branch digunakan untuk workflow development
8. `main` harus selalu dalam kondisi stabil
9. Jika ragu, commit dulu sebelum eksperimen
10. Kehilangan 1 commit lebih baik daripada kehilangan 8 jam kerja

---

# Final Workflow

```text
main
│
└── develop
     │
     ├── feature/auth
     ├── feature/product
     ├── feature/customer
     ├── feature/transaction
     └── feature/report
```

Flow:

```text
feature/* -> develop -> main
```

Atau untuk solo developer:

```text
develop -> main
```

Sudah cukup untuk sebagian besar kebutuhan development aplikasi POS skala kecil sampai menengah.
