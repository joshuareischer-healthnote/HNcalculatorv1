# HNcalculatorv1
Version1

from flask import Flask, request, render_template_string

app = Flask(__name__)

def calculate_total_cost(provider_count):
    tiers = [
        (0, 25, 225),
        (26, 50, 187.50),
        (51, 100, 137.50),
        (101, 250, 112.50),
        (251, 500, 87.50),
        (501, float('inf'), 62.50)
    ]
    
    total_cost = 0
    remaining_providers = provider_count
    
    for lower, upper, rate in tiers:
        if remaining_providers <= 0:
            break
        
        num_in_tier = min(remaining_providers, upper - lower + 1)
        annual_rate_per_provider = rate * 12
        total_cost += num_in_tier * annual_rate_per_provider
        remaining_providers -= num_in_tier
    
    return total_cost

@app.route("/", methods=["GET", "POST"])
def index():
    result = None
    if request.method == "POST":
        try:
            provider_count = int(request.form["provider_count"])
            if provider_count < 0:
                raise ValueError("Provider count cannot be negative.")
            result = f"Total Annual Cost: ${calculate_total_cost(provider_count):,.2f}"
        except ValueError as e:
            result = f"Invalid input: {e}"
    
    return render_template_string("""
    <!doctype html>
    <html>
    <head>
        <title>Cost Calculator</title>
        <style>
            body { font-family: Arial, sans-serif; text-align: center; padding: 50px; }
            form { margin: 20px auto; width: 300px; }
            input, button { padding: 10px; margin: 5px; width: 100%; }
        </style>
    </head>
    <body>
        <h2>Provider Cost Calculator</h2>
        <form method="post">
            <label for="provider_count">Enter Number of Providers:</label>
            <input type="number" name="provider_count" required>
            <button type="submit">Calculate</button>
        </form>
        {% if result %}
            <h3>{{ result }}</h3>
        {% endif %}
    </body>
    </html>
    """, result=result)

if __name__ == "__main__":
    app.run(debug=True)
