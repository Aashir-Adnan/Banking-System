# Banking System — ASP.NET Web Forms + SQL Server

An online banking web application built with **ASP.NET Web Forms (C#, .NET Framework 4.7.2)** on a **Microsoft SQL Server** database managed through SSMS. It was written as a university Database Systems project. It has two portals:

- a **customer portal**: sign-up requests, login, deposits, withdrawals, transfers, bill payments, loan applications, heir (nominee) registration, complaints, and transaction history;
- an **admin portal**: approving sign-up requests, freezing and unfreezing accounts, and reviewing loans, transactions and complaints.

The full Visual Studio project is in **`DBProj.zip`**. The original notes are in `README.txt`.

---

## Table of contents

- [Repository contents](#repository-contents)
- [Features](#features)
  - [Customer portal](#customer-portal)
  - [Admin portal](#admin-portal)
  - [Static / informational pages](#static--informational-pages)
- [Tech stack](#tech-stack)
- [Project layout (inside `DBProj.zip`)](#project-layout-inside-dbprojzip)
- [Database](#database)
  - [Tables referenced by the code](#tables-referenced-by-the-code)
  - [Stored procedures referenced by the code](#stored-procedures-referenced-by-the-code)
- [Getting started](#getting-started)
- [Configuration](#configuration)
- [How it works](#how-it-works)
- [Troubleshooting](#troubleshooting)
- [Known limitations](#known-limitations)
- [Author](#author)

---

## Repository contents

| File | Description |
|------|-------------|
| `DBProj.zip` | The complete Visual Studio solution (`DBProj.csproj`, every `.aspx` page with its `.aspx.cs` code-behind and `.designer.cs` files, `Web.config`, stylesheet, favicon) |
| `README.txt` | Original setup notes |

---

## Features

### Customer portal

| Page | Purpose | Data access |
|------|---------|-------------|
| `SignUp.aspx` | Request a new account (full name, CNIC, username, PIN, account type). It creates a *request*, not an account. An admin has to approve it. | `InsertIntoSignUp` stored procedure |
| `Login.aspx` | Log in with username + PIN. The same page also handles admin login against the `ADMINS` table. | `SELECT UserID FROM Accounts WHERE Username=@username AND Pin=@Pin` |
| `Welcome.aspx` | Customer landing page after login | — |
| `Details.aspx` | Account summary: name, CNIC, balance, account type, status, heirs, plus the user's cards (type, credit limit) | `Accounts`, `Cards` |
| `Deposit.aspx` | Deposit money | `EXEC DEPOSIT @Amount, @UserID` |
| `Withdrawal.aspx` | Withdraw money | `EXEC WITHDRAWAL @Amount, @UserID` |
| `Transfer.aspx` | Send money to another account after re-entering the PIN | PIN re-check, then `InsertIntoTransactions` |
| `Bills.aspx` | Pay a bill to a receiver and list past bills | PIN re-check, `InsertIntoBills`, `SELECT … FROM Bills` |
| `Loan.aspx` | Apply for a loan | PIN re-check, then `InsertIntoLoans` |
| `Heir.aspx` | Register an heir/nominee by name and CNIC | `InsertIntoHeirs` |
| `Incoming.aspx` | Received transactions (sender name, amount, date) | `Transactions JOIN Accounts ON SenderID` |
| `Outgoing.aspx` | Sent transactions (receiver name, amount, date) | `Transactions JOIN Accounts ON ReceiverID` |
| `Complaint.aspx` | Submit a complaint and see the 10 most recent ones | `InsertIntoComplaints`, `SELECT TOP 10 …` |

### Admin portal

| Page | Purpose |
|------|---------|
| `AdminWelcome.aspx` | Dashboard showing the total number of accounts and pending sign-ups, a paged list of accounts, and **freeze/unfreeze** (`UPDATE Accounts SET AccountStatus = 0/1`) |
| `AdminSignUp.aspx` | Paged list of sign-up requests. Approving one sets `ReqStatus = 1`. |
| `AdminLoan.aspx` | Paged list of loans. Admins can delete/resolve a loan (`DELETE FROM Loans`). |
| `AdminTran.aspx` | Paged list of every transaction in the bank |
| `AdminComplaint.aspx` | The 10 most recent complaints, with delete/resolve (`DELETE FROM Complaints`) |

All admin lists page on the server with `ORDER BY … OFFSET @Offset ROWS FETCH NEXT @PageSize ROWS ONLY`.

### Static / informational pages

`Homepage.aspx`, `AboutUs.aspx`, `Contact.aspx`, `News.aspx`, `Accessibility.aspx`, `PrivacyPolicy.aspx`, `TermsOfService.aspx`. They are styled by `StyleSheet1.css`.

---

## Tech stack

| Layer | Technology |
|-------|------------|
| Web framework | ASP.NET Web Forms, .NET Framework **4.7.2** |
| Language | C# (code-behind) |
| Data access | ADO.NET (`SqlConnection`, `SqlCommand`, parameterised queries, stored procedures) |
| Database | Microsoft SQL Server (Express or Developer), administered through SQL Server Management Studio |
| UI | `.aspx` markup, `asp:GridView` / `asp:TextBox` / `asp:Button` server controls, a custom CSS file |
| Compiler package | `Microsoft.CodeDom.Providers.DotNetCompilerPlatform` 2.0.1 |

---

## Project layout (inside `DBProj.zip`)

```
DBProj/
├── DBProj.csproj               # Visual Studio Web Application project
├── Web.config                  # Connection string + compilation settings
├── Web.Debug.config / Web.Release.config
├── packages.config
├── StyleSheet1.css             # Shared styles
├── favicon.ico / favicon.png
├── Properties/AssemblyInfo.cs
│
├── Homepage.aspx  AboutUs.aspx  Contact.aspx  News.aspx
├── Accessibility.aspx  PrivacyPolicy.aspx  TermsOfService.aspx
│
├── Login.aspx(.cs)             # Customer + admin login
├── SignUp.aspx(.cs)            # Account request
├── Welcome.aspx                # Customer home
├── Details.aspx(.cs)           # Account + cards
├── Deposit.aspx(.cs)   Withdrawal.aspx(.cs)   Transfer.aspx(.cs)
├── Bills.aspx(.cs)     Loan.aspx(.cs)         Heir.aspx(.cs)
├── Incoming.aspx(.cs)  Outgoing.aspx(.cs)     Complaint.aspx(.cs)
│
└── AdminWelcome.aspx(.cs)  AdminSignUp.aspx(.cs)  AdminLoan.aspx(.cs)
    AdminTran.aspx(.cs)     AdminComplaint.aspx(.cs)
```

Each `.aspx` page has a matching `.aspx.cs` code-behind file with the event handlers, and a generated `.aspx.designer.cs` file with the control declarations.

---

## Database

The database is named **`DB Project`** (see the connection string). **The repository has no SQL schema script.** You have to create the database yourself, or restore it from your own backup. The tables and procedures below are the ones the C# code expects.

### Tables referenced by the code

| Table | Columns used by the application |
|-------|--------------------------------|
| `Accounts` | `UserID`, `FullName`, `CNIC`, `Username`, `Pin`, `Balance`, `AccountType`, `AccountStatus` (1 = active, 0 = frozen), `Heirs` |
| `ADMINS` | `UserID`, `Username`, `Pin` |
| `SignUp` | `RequestID`, `FullName`, `CNIC`, `Username`, `Pin`, `AccountType`, `ReqStatus` (1 = approved) |
| `Transactions` | `TransactionID`, `SenderID`, `ReceiverID`, `Amount`, `Date` |
| `Bills` | `BillID`, `UserID`, `ReceiverID`, `TransactionAmount`, `Date` |
| `Loans` | `LoanID`, … (shown as-is in the admin grid) |
| `Cards` | `CardID`, `UserID`, `CardType`, `CreditLimit` |
| `Complaints` | `ComplainID`, `Complain` |

### Stored procedures referenced by the code

| Procedure | Called from | Expected behaviour |
|-----------|-------------|--------------------|
| `DEPOSIT @Amount, @UserID` | `Deposit.aspx.cs` | Adds `@Amount` to the user's balance |
| `WITHDRAWAL @Amount, @UserID` | `Withdrawal.aspx.cs` | Subtracts `@Amount` from the balance |
| `InsertIntoTransactions` | `Transfer.aspx.cs` | Records a transfer and moves the balance from sender to receiver |
| `InsertIntoBills` | `Bills.aspx.cs` | Records a bill payment |
| `InsertIntoLoans` | `Loan.aspx.cs` | Records a loan application |
| `InsertIntoHeirs` | `Heir.aspx.cs` | Adds an heir (`@HeirName`, `@HeirCNIC`) |
| `InsertIntoComplaints` | `Complaint.aspx.cs` | Adds a complaint (`@Complain`) |
| `InsertIntoSignUp` | `SignUp.aspx.cs` | Adds an account request |

Approving a request in `AdminSignUp` only sets `ReqStatus = 1`. If approval is supposed to create the matching `Accounts` row, a trigger on `SignUp` (or a manual step) has to do it.

---

## Getting started

### Prerequisites

- Windows with **Visual Studio 2019/2022** and the *ASP.NET and web development* workload
- **.NET Framework 4.7.2** developer pack
- **SQL Server** (Express is enough) and **SQL Server Management Studio**

### Steps

1. **Clone and unzip**

   ```bash
   git clone https://github.com/Aashir-Adnan/Banking-System.git
   ```

   Extract `DBProj.zip` and open `DBProj/DBProj.csproj` in Visual Studio. It will offer to create a solution file.
2. **Create the database** `DB Project` in SSMS, with the tables and stored procedures listed [above](#database). Add at least one row to `ADMINS` so you can log in to the admin portal.
3. **Point the app at your server.** Edit `Web.config` ([see below](#configuration)).
4. **Restore NuGet packages.** Right-click the solution and choose *Restore NuGet Packages*.
5. **Run** with **F5** (IIS Express). Start at `Homepage.aspx` or `Login.aspx`.

---

## Configuration

The only setting is the connection string named `myConnectionstring` in `Web.config`:

```xml
<connectionStrings>
  <add name="myConnectionstring"
       connectionString="Data Source=LAPTOP-VDL5E083;Initial Catalog= DB Project; Integrated Security=True"/>
</connectionStrings>
```

Change `Data Source` to your own SQL Server instance, for example `.\SQLEXPRESS` or `(localdb)\MSSQLLocalDB`. With `Integrated Security=True` the app connects as your Windows user.

---

## How it works

- **Session handling.** `Login.aspx.cs` stores the logged-in user's ID in the static properties `Login.Login_ID` and `Login.AdminLogin_ID`, and every other page reads them from there. Opening `Login.aspx` resets `Login_ID` to 0. If a page finds no logged-in user, it redirects to `Login.aspx`.
- **PIN confirmation.** Sensitive actions (transfer, bill payment, loan) ask for the PIN again and check it with `SELECT UserID FROM Accounts WHERE UserID=@UserID AND Pin=@Pin` before calling the stored procedure.
- **Parameterised SQL.** Every query passes values with `SqlCommand.Parameters.AddWithValue`, which protects against SQL injection. Money-moving logic lives in stored procedures, so it can be atomic on the database side.
- **Paging.** Admin grids page on the server with `OFFSET/FETCH`. You can switch to `GridView`'s built-in paging instead (see [Troubleshooting](#troubleshooting)).

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `A network-related or instance-specific error…` | `Data Source` in `Web.config` doesn't match your SQL Server instance name |
| `Could not find stored procedure 'DEPOSIT'` (or similar) | The procedure hasn't been created in `DB Project` (see [Stored procedures](#stored-procedures-referenced-by-the-code)) |
| Paging in the admin pages misbehaves | Switch the grid to automatic paging: `<asp:GridView … AllowPaging="true" PageSize="10">` |
| Missing `roslyn\csc.exe` on build | Restore NuGet packages, then rebuild |

---

## Known limitations

This is an academic project and should not be used for real banking:

- **Static login state.** `Login_ID` is a `static` property, so it's shared by everyone using the same app instance. Two people logged in at once would overwrite each other's session. A production version would use `Session[...]` or ASP.NET Identity.
- **PINs are stored and compared in plain text.**
- There is no HTTPS enforcement, CSRF protection or rate limiting on login.
- No database schema or seed script is included.

---

## Author

**Aashir Adnan**: [GitHub @Aashir-Adnan](https://github.com/Aashir-Adnan)
