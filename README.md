# Concurrent-Banking-System

Système bancaire concurrent (mini core banking)

Ce dépôt contient un projet **.NET 8 (C#)** complet :
- Backend minimal API
- Tests de robustesse
- Logger d'audit asynchrone
- Détecteur de fraude en thread séparé

Le but est de simuler des transactions concurrentes entre comptes.

---

## 🔹 Conception principale

- **Accounts in-memory** : `(id: long, balance: long en centimes)`
- **Transferts atomiques** entre comptes
- **Verrouillage fin** avec lock hiérarchisé par ID de compte (évite les deadlocks)
- **Audit logging asynchrone** via une queue (ne bloque pas les transactions)
- **Tests de charge multi-thread** (milliers de transactions) vérifiant l’intégrité (somme totale constante)
- **Détecteur de fraude** qui scanne les transactions en temps réel

---

## 📂 Arborescence du projet

```plaintext
ConcurrentBankingSystem/
├─ src/
│  ├─ ConcurrentBanking.Api/         (ASP.NET Core minimal API)
│  │  ├─ Program.cs
│  │  ├─ Models/Account.cs
│  │  ├─ Models/DTOs.cs
│  │  ├─ Repositories/IAccountRepository.cs
│  │  ├─ Repositories/InMemoryAccountRepository.cs
│  │  ├─ Services/BankService.cs
│  │  ├─ Services/AuditLogger.cs
│  │  ├─ Services/FraudDetector.cs
│  │  └─ appsettings.json
│  └─ ConcurrentBanking.LoadTest/    (Console test runner)
│     └─ Program.cs
├─ tests/
│  └─ ConcurrentBanking.Tests/
│     └─ BankTests.cs
├─ docker-compose.yml
├─ Dockerfile
└─ README.md
```
---

## ⚙️ Choix techniques & raisons

- **Langage** : C# (.NET 8) — usage de `Interlocked` et `Monitor`
- **Atomicité** :
    - `Interlocked` pour les opérations simples
    - `Monitor` (`lock` / `Monitor.Enter`) pour les sections critiques
- **Lock hiérarchisé** : acquisition des verrous par ordre croissant d’`accountId` (évite les deadlocks)
- **Audit** : `BlockingCollection<LogEntry>` consommée par un worker → écrit vers console/fichier (asynchrone, non bloquant)
- **Tests de charge** : `Task.Run` + `Parallel` + `SemaphoreSlim` → génèrent des milliers de transactions
- **Fraude** : thread séparé lisant le flux d’audit et détectant transactions suspectes (seuil/pattern)

---

## 🚀 Installation (prérequis)

- [.NET 8 SDK](https://dotnet.microsoft.com/en-us/download/dotnet/8.0)
- [Docker](https://www.docker.com/)

---

## ▶️ Lancer localement

### API
```bash
cd src/ConcurrentBanking.Api
dotnet run

➡️ L’API démarre sur http://localhost:5000

cd src/ConcurrentBanking.LoadTest
dotnet run
```

🌐 API Endpoints

- Récupérer tous les comptes

curl http://localhost:5000/accounts

- Récupérer un compte par ID

curl http://localhost:5000/accounts/1

- Créer un compte

curl -X POST http://localhost:5000/accounts \
-H "Content-Type: application/json" \
-d '{"id": 1, "balance": 10000}'

- Effectuer un transfert

curl -X POST http://localhost:5000/transfer \
-H "Content-Type: application/json" \
-d '{"fromAccountId": 1, "toAccountId": 2, "amount": 500}'


🔹 Audit & Fraude

- Récupérer les logs d’audit

curl http://localhost:5000/audit

- Récupérer les alertes de fraude

curl http://localhost:5000/fraud
