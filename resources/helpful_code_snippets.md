**Helpful Code Blocks**

I have added some blocks below that will help you in building out your web application.  Before you try using these blocks, keep in mind that these code block examples are using some specific 'example vocabulary':

 - The example database table is named entries with columns:
   - id (INTEGER PRIMARY KEY AUTOINCREMENT)
   - title (TEXT)
   - content (TEXT)
   - user (TEXT for username).

 - Users must be logged in; the current user is stored in session["user"].
 - All CRUD queries filter by user=? to ensure users only access their own data.
 - Routes are:
    - /dashboard (list entries)
    - /create (add entry)
    - /edit/<id> (edit entry)
    - /delete/<id> (delete entry)
 - Entry placeholders use {{ entry['title'] }} and {{ entry['content'] }}.
 - A get_db() helper returns an SQLite connection; always close the connection after queries.
 - Security: never allow editing or deleting without filtering by user.
 - Entry IDs are integers passed in URLs (<int:id>).
 - The table can be extended (e.g., date, tags) but CRUD queries must be updated accordingly.

---

1. Add a new database to your application:
```
conn.execute("""
    CREATE TABLE IF NOT EXISTS entries (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT,
        content TEXT,
        user TEXT
    )
""")
```

---
2. Protect routes so only users who are logged in can access:

```
def require_login():
    if "user" not in session:
        return False
    return True
```

Example Usage
```
if not require_login():
    return redirect(url_for("login"))
```
---
3. Show data (note the session reference here for security):

```
@app.route("/dashboard")
def dashboard():
    if "user" not in session:
        return redirect(url_for("login"))

    conn = get_db()
    entries = conn.execute(
        "SELECT * FROM entries WHERE user=?",
        (session["user"],)
    ).fetchall()
    conn.close()

    page = f"""{base_style}
    <div class="card">
    <h2>Your Entries</h2>

    <a href="/create"><button>Create New</button></a>
    <br><br>

    {% for entry in entries %}
        <div>
            <h3>{{ entry['title'] }}</h3>
            <p>{{ entry['content'] }}</p>
            <a href="/edit/{{ entry['id'] }}">Edit</a>
            <a href="/delete/{{ entry['id'] }}">Delete</a>
            <hr>
        </div>
    {% endfor %}

    <a href="/logout"><button>Logout</button></a>
    </div>
    """

    return render_template_string(page, entries=entries)
```
Example Usage
```
return redirect(url_for("dashboard"))
```
---
4. Create New Data

```
@app.route("/create", methods=["GET", "POST"])
def create():
    if "user" not in session:
        return redirect(url_for("login"))

    if request.method == "POST":
        title = request.form["title"]
        content = request.form["content"]

        conn = get_db()
        conn.execute(
            "INSERT INTO entries (title, content, user) VALUES (?, ?, ?)",
            (title, content, session["user"])
        )
        conn.commit()
        conn.close()

        return redirect(url_for("dashboard"))

    page = f"""{base_style}
    <div class="card">
    <h2>New Entry</h2>
    <form method="POST">
        <input name="title" placeholder="Title"><br>
        <input name="content" placeholder="Content"><br>
        <button type="submit">Save</button>
    </form>
    </div>
    """
    return render_template_string(page)
```
---
5. Edit Data

```
@app.route("/edit/<int:id>", methods=["GET", "POST"])
def edit(id):
    if "user" not in session:
        return redirect(url_for("login"))

    conn = get_db()

    entry = conn.execute(
        "SELECT * FROM entries WHERE id=? AND user=?",
        (id, session["user"])
    ).fetchone()

    if not entry:
        conn.close()
        return "Not allowed"

    if request.method == "POST":
        title = request.form["title"]
        content = request.form["content"]

        conn.execute(
            "UPDATE entries SET title=?, content=? WHERE id=?",
            (title, content, id)
        )
        conn.commit()
        conn.close()

        return redirect(url_for("dashboard"))

    page = f"""{base_style}
    <div class="card">
    <h2>Edit Entry</h2>
    <form method="POST">
        <input name="title" value="{{ entry['title'] }}"><br>
        <input name="content" value="{{ entry['content'] }}"><br>
        <button type="submit">Update</button>
    </form>
    </div>
    """
    return render_template_string(page, entry=entry)
```
---
6. Remove Data

```
@app.route("/delete/<int:id>")
def delete(id):
    if "user" not in session:
        return redirect(url_for("login"))

    conn = get_db()
    conn.execute(
        "DELETE FROM entries WHERE id=? AND user=?",
        (id, session["user"])
    )
    conn.commit()
    conn.close()

    return redirect(url_for("dashboard"))
```
