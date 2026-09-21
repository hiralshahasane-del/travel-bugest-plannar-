<!DOCTYPE html>
<html>
<head>
    <title>Travel Budget Planner</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>

<body>

<div class="container">
    <h1>✈️ Travel Budget Planner</h1>

    <form method="POST">

        <label>Destination</label>
        <input type="text" name="destination" required>

        <label>Number of Days</label>
        <input type="number" name="days" required>

        <label>Transport Cost</label>
        <input type="number" name="transport" required>

        <label>Hotel Cost</label>
        <input type="number" name="hotel" required>

        <label>Food Cost</label>
        <input type="number" name="food" required>

        <label>Activities Cost</label>
        <input type="number" name="activities" required>

        <button type="submit">Calculate Budget</button>
    </form>

    {% if result %}
    <div class="result">
        <h2>Budget Summary</h2>
        <p>Destination: {{ result.destination }}</p>
        <p>Total Budget: ₹{{ result.total }}</p>
        <p>Daily Budget: ₹{{ result.daily }}</p>
    </div>
    {% endif %}

</div>
body {
    font-family: Arial, sans-serif;
    background: #f2f6f8;
}

.container {
    width: 500px;
    margin: 50px auto;
    padding: 30px;
    background: white;
    border-radius: 15px;
}

h1 {
    text-align: center;
}

label {
    display: block;
    margin-top: 15px;
}

input {
    width: 100%;
    padding: 10px;
    margin-top: 5px;
    box-sizing: border-box;
}

button {
    width: 100%;
    padding: 12px;
    margin-top: 20px;
    cursor: pointer;
}

.result {
    margin-top: 25px;
    padding: 15px;
    border-radius: 10px;
}
from flask import Flask, render_template, request

app = Flask(__name__)

@app.route("/", methods=["GET", "POST"])
def home():
    result = None

    if request.method == "POST":
        destination = request.form["destination"]
        days = int(request.form["days"])
        transport = float(request.form["transport"])
        hotel = float(request.form["hotel"])
        food = float(request.form["food"])
        activities = float(request.form["activities"])

        total = transport + hotel + food + activities
        daily_budget = total / days

        result = {
            "destination": destination,
            "total": total,
            "daily": daily_budget
        }

    return render_template("index.html", result=result)

if __name__ == "__main__":
    app.run(debug=True)



</body>
</html>
