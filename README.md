# travel-bugest-plannar-
"""
Travel Budget Planner
A simple Flask + SQLite + HTML/CSS web app to plan and track
travel expenses against a total budget.

Run with:  python app.py
Then open: http://127.0.0.1:5000
"""

from flask import Flask, render_template, request, redirect, url_for, flash
import sqlite3
import os

app = Flask(__name__)
app.secret_key = "travel-budget-planner-secret"

DB_PATH = os.path.join(os.path.dirname(__file__), "budget.db")

CATEGORIES = ["Transport", "Accommodation", "Food", "Activities", "Miscellaneous"]


def get_db():
    conn = sqlite3.connect(DB_PATH)
    conn.row_factory = sqlite3.Row
    return conn


def init_db():
    conn = get_db()
    conn.execute("""
        CREATE TABLE IF NOT EXISTS budget (
            id INTEGER PRIMARY KEY CHECK (id = 1),
            amount REAL NOT NULL
        )
    """)
    conn.execute("""
        CREATE TABLE IF NOT EXISTS expenses (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            category TEXT NOT NULL,
            description TEXT,
            amount REAL NOT NULL,
            date_added TEXT DEFAULT CURRENT_TIMESTAMP
        )
    """)
    conn.commit()
    conn.close()


def get_budget():
    conn = get_db()
    row = conn.execute("SELECT amount FROM budget WHERE id = 1").fetchone()
    conn.close()
    return row["amount"] if row else None


@app.route("/")
def index():
    budget = get_budget()
    conn = get_db()
    expenses = conn.execute(
        "SELECT * FROM expenses ORDER BY date_added DESC"
    ).fetchall()
    conn.close()

    spent = sum(e["amount"] for e in expenses)
    remaining = (budget - spent) if budget is not None else None
    percent_used = round((spent / budget) * 100, 1) if budget else 0

    # category-wise totals for the simple bar breakdown
    category_totals = {c: 0 for c in CATEGORIES}
    for e in expenses:
        category_totals[e["category"]] = category_totals.get(e["category"], 0) + e["amount"]
    max_cat = max(category_totals.values()) if category_totals.values() else 0

    return render_template(
        "index.html",
        budget=budget,
        expenses=expenses,
        spent=spent,
        remaining=remaining,
        percent_used=percent_used,
        categories=CATEGORIES,
        category_totals=category_totals,
        max_cat=max_cat,
    )


@app.route("/set_budget", methods=["POST"])
def set_budget():
    try:
        amount = float(request.form["amount"])
        if amount < 0:
            raise ValueError
    except (ValueError, KeyError):
        flash("Please enter a valid budget amount.", "error")
        return redirect(url_for("index"))

    conn = get_db()
    conn.execute("INSERT OR REPLACE INTO budget (id, amount) VALUES (1, ?)", (amount,))
    conn.commit()
    conn.close()
    flash("Budget updated.", "success")
    return redirect(url_for("index"))


@app.route("/add_expense", methods=["POST"])
def add_expense():
    category = request.form.get("category", "Miscellaneous")
    description = request.form.get("description", "").strip()
    try:
        amount = float(request.form["amount"])
        if amount <= 0:
            raise ValueError
    except (ValueError, KeyError):
        flash("Please enter a valid expense amount.", "error")
        return redirect(url_for("index"))

    conn = get_db()
    conn.execute(
        "INSERT INTO expenses (category, description, amount) VALUES (?, ?, ?)",
        (category, description, amount),
    )
    conn.commit()
    conn.close()
    flash("Expense added.", "success")
    return redirect(url_for("index"))


@app.route("/delete_expense/<int:expense_id>", methods=["POST"])
def delete_expense(expense_id):
    conn = get_db()
    conn.execute("DELETE FROM expenses WHERE id = ?", (expense_id,))
    conn.commit()
    conn.close()
    flash("Expense removed.", "success")
    return redirect(url_for("index"))


@app.route("/reset", methods=["POST"])
def reset():
    conn = get_db()
    conn.execute("DELETE FROM expenses")
    conn.execute("DELETE FROM budget")
    conn.commit()
    conn.close()
    flash("Planner reset.", "success")
    return redirect(url_for("index"))


if __name__ == "__main__":
    init_db()
    app.run(debug=True)
