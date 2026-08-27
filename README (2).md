Maisha - Login & Registration System (SQL Server)

## What the application does

This is a desktop C# WinForms application with three screens: a login form, a
registration form, and a dashboard shown after a successful login. Users can
register a new account, log in with that account, and log out back to the
login screen. All account data (username and password) is stored in a SQL
Server table instead of a local Microsoft Access file.

## How to run it

1. Open SQL Server Management Studio (or the SQL Server Object Explorer in
   Visual Studio) and run `database.sql` from this repo. It creates the
   `db_users` database, the `tbl_users` table, and one test account.
2. Open the solution in Visual Studio and build it (Ctrl+Shift+B).
3. If your SQL Server instance isn't LocalDB, open `App.config` and change
   only the `Data Source=` part of the connection string — for example
   `Data Source=localhost;` or `Data Source=DESKTOP-XXXXX\SQLEXPRESS;`.
   Leave everything else in the connection string alone.
4. Run the app. Test login: **admin / admin123**.

## What I changed, and why

**Files edited:** `App.config`, `frmLogin.cs`, `frmRegister.cs`,
`frmDashboard.cs`, `Program.cs`.

**Why I replaced OleDb.** The original app connected to a Microsoft Access
file (`db_users.mdb`) through `System.Data.OleDb`, using a hard-coded
`Provider=Microsoft.Jet.OLEDB.4.0` connection string in each form. Access
files like this are fragile — they only work if the `.mdb` sits in exactly
the right folder relative to the `.exe`, the Jet/ACE driver has to be
installed separately on whatever machine runs it, and it's not something
this course actually teaches. I replaced `OleDbConnection`/`OleDbCommand`
with `SqlConnection`/`SqlCommand` from `System.Data.SqlClient`, pointing at
a real `tbl_users` table in SQL Server.

**Why the connection string lives in `App.config` instead of inside each
form.** The original code built a new `OleDbConnection` with the connection
string typed directly into `frmLogin.cs`. If the database ever moved (a
different server name, a different machine), every form that touched the
database would need to be edited and recompiled individually. Keeping the
one connection string in `App.config` and reading it with
`ConfigurationManager.ConnectionStrings["connString"].ConnectionString`
means there's exactly one place to change it, and the compiled `.exe`
doesn't need to be rebuilt if the server changes — only the config file
next to it does.

**What `@username` and `@password` are for.** The original login query was
built by string concatenation:
```
"... WHERE username = '" + txtUsername.Text + "' and password = '" + txtPassword.Text + "'"
```
Typing `' OR '1'='1` into the password box turns that into a query that's
always true, logging you in as the first row in the table with no valid
password at all — that's SQL injection. `@username` and `@password` are
parameters: the value the user typed is sent to SQL Server separately from
the query text, so it's always treated as *data*, never as part of the SQL
statement itself. That closes off the injection entirely.

**Logout.** The original `btnLogout_Click` called `Application.Exit()` and
displayed "Goodbye Sayan" — that quits the whole program, it isn't a
logout. The fixed version opens a new `frmLogin` and closes the dashboard,
so the user actually lands back on the login screen and can log in again
without restarting the app.

