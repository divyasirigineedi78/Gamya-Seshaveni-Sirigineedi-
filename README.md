from flask import Flask, render_template, request, redirect, session
import sqlite3

app = Flask(__name__)
app.secret_key = "ecommerce"

def connect():
    conn = sqlite3.connect("database.db")
    conn.row_factory = sqlite3.Row
    return conn

@app.route("/")
def home():
    conn = connect()
    products = conn.execute("SELECT * FROM products").fetchall()
    conn.close()
    return render_template("index.html", products=products)

@app.route("/register", methods=["GET","POST"])
def register():
    if request.method == "POST":
        username = request.form["username"]
        password = request.form["password"]

        conn = connect()
        conn.execute(
            "INSERT INTO users(username,password) VALUES(?,?)",
            (username,password)
        )
        conn.commit()
        conn.close()

        return redirect("/login")

    return render_template("register.html")

@app.route("/login", methods=["GET","POST"])
def login():
    if request.method == "POST":
        username = request.form["username"]
        password = request.form["password"]

        conn = connect()
        user = conn.execute(
            "SELECT * FROM users WHERE username=? AND password=?",
            (username,password)
        ).fetchone()
        conn.close()

        if user:
            session["user"] = username
            return redirect("/")

    return render_template("login.html")

@app.route("/add/<int:id>")
def add(id):
    if "cart" not in session:
        session["cart"] = []

    cart = session["cart"]
    cart.append(id)
    session["cart"] = cart

    return redirect("/")

@app.route("/cart")
def cart():
    if "cart" not in session:
        session["cart"] = []

    conn = connect()

    products = []

    for pid in session["cart"]:
        p = conn.execute(
            "SELECT * FROM products WHERE id=?",
            (pid,)
        ).fetchone()

        if p:
            products.append(p)

    conn.close()

    return render_template("cart.html", products=products)

@app.route("/checkout")
def checkout():
    session["cart"] = []
    return "Order Placed Successfully!"

if __name__ == "__main__":
    app.run(debug=True)
