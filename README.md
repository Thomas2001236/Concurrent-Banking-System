# Concurrent-Banking-System

Système bancaire concurrent (mini core banking)

Ce dépôt contient un projet .NET 8 (C#) complet — backend, tests de robustesse, logger d'audit asynchrone et détecteur de fraude en thread séparé — conçu pour simuler des transactions concurrentes entre comptes.

Conception principale

Accounts in-memory (id long, balance long in cents).

Transferts atomiques entre comptes.

Verrouillage fin avec lock hiérarchisé par ID de compte (pour éviter deadlocks).

Audit logging asynchrone via queue (ne bloque pas les transactions).

Tests de charge multi-thread (milliers de transactions) qui vérifient l'intégrité (somme totale constante).

Détecteur de fraude qui scanne les transactions pendant l'exécution.

Arborescence du projet
ConcurrentBankingSystem/
├─ src/
│  ├─ ConcurrentBanking.Api/    (ASP.NET Core minimal API)
│  │  ├─ Program.cs
│  │  ├─ Models/Account.cs
│  │  ├─ Models/DTOs.cs
│  │  ├─ Repositories/IAccountRepository.cs
│  │  ├─ Repositories/InMemoryAccountRepository.cs
│  │  ├─ Services/BankService.cs
│  │  ├─ Services/AuditLogger.cs
│  │  ├─ Services/FraudDetector.cs
│  │  └─ appsettings.json
│  └─ ConcurrentBanking.LoadTest/ (Console test runner)
│     └─ Program.cs
├─ tests/
│  └─ ConcurrentBanking.Tests/
│     └─ BankTests.cs
├─ docker-compose.yml
├─ Dockerfile
└─ README.md
Choix techniques & raisons

Langage : C# (.NET 8). utilisation de Interlocked et Monitor — d'ou le choix de C#.

Atomicité : on utilise Interlocked pour opérations simples et Monitor (via lock / Monitor.Enter) pour sections critiques lors de transferts entre 2 comptes.

Lock hiérarchisé : toujours acquérir les verrous dans l'ordre croissant d'accountId pour éviter deadlocks.

Audit : une BlockingCollection<LogEntry> consommée par un worker pour écrire vers console / fichier (asynchrone, non bloquant pour la logique métier).

Tests de charge : Task.Run + Parallel + SemaphoreSlim pour générer des milliers de transactions.

Fraude : thread séparé qui lit le flux d'audit et alerte quand une transaction dépasse un seuil ou pattern suspect.

Installation (prérequis)

.NET 8 SDK installé

Docker

Lancer localement

cd src/ConcurrentBanking.Api

dotnet run (API démarrera sur http://localhost:5000 par défaut)

Dans un autre terminal, cd src/ConcurrentBanking.LoadTest puis dotnet run pour lancer la simulation de transactions.
