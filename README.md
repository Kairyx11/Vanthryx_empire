# Vanthryx_empire
"Mobile-powered AI command system for business deployment"
# ============================================
# COMPLETE BUSINESS ADMIN PANEL - ONE SCRIPT
# WITH .gitignore RECOMMENDATIONS INCLUDED AS COMMENTS
# ============================================

import os
import json
from flask import Flask, render_template_string, request, redirect, session, flash
import secrets
from datetime import datetime

# === .gitignore recommendations (for your repo) ===
"""
# OS Specific
.DS_Store
Thumbs.db
ehthumbs.db
Icon?
Desktop.ini

# Python
__pycache__/
*.py[cod]
*.pyo
*.pyd
.Python
env/
venv/
ENV/
*.env
*.env.*

# Node / React / Electron
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*
package-lock.json
.pnp.*
.pnp.js

# Build Artifacts
dist/
build/
out/
*.log
*.exe
*.dll
*.app
*.so
*.deb
*.rpm
*.snap
*.img
*.iso
*.apk
*.tar.gz
*.zip

# IDE Configs
.vscode/
.idea/
*.swp
*.swo

# Git
*.orig

# Custom Vault / Secure Files
*.key
*.pem
*.crt
vault.json
*.sqlite
*.db
auth.json
token.txt

# Backup Files
*.bak
*.old
*.tmp
*.log

# Vanthryx Empire Launcher
/launcher_config.json
/secret_ai_core/
/offline_wallet_sync/
/ghostmode_boot/
"""

# === Flask App ===
app = Flask(__name__)
app.secret_key = secrets.token_hex(32)

# --- Simple in-memory storage ---
CUSTOMERS = []
REVENUE = 0
ADMIN_PASSWORD = "Ka1ryxbr1dg3"  # CHANGE THIS!

# --- Security Headers ---
@app.after_request
def add_security_headers(response):
    response.headers['X-Content-Type-Options'] = 'nosniff'
    response.headers['X-Frame-Options'] = 'DENY'
    response.headers['X-XSS-Protection'] = '1; mode=block'
    return response

# --- Input validation ---
def validate_input(data):
    return bool(data and len(data) >= 3)

# --- HTML Templates ---
HOME_PAGE = """
<!DOCTYPE html>
<html>
<head>
    <title>Business Solutions</title>
    <style>
        body { font-family: Arial; margin: 50px; background: #f4f4f4; }
        .container { background: white; padding: 30px; border-radius: 10px; }
        .price-box { border: 2px solid #007bff; padding: 20px; margin: 10px; display: inline-block; }
        button { background: #007bff; color: white; padding: 10px 20px; border: none; cursor: pointer; }
        button:hover { background: #0056b3; }
    </style>
</head>
<body>
    <div class="container">
        <h1>🚀 Professional Business Solutions</h1>
        <p>Transform your business with our admin panel solutions!</p>
        <h2>💰 Our Services</h2>
        <div class="price-box">
            <h3>Basic Setup - $500</h3>
            <p>✅ Custom admin panel</p>
            <p>✅ User management</p>
            <p>✅ Basic security</p>
            <button onclick="buyService('basic')">Buy Now</button>
        </div>
        <div class="price-box">
            <h3>Pro Package - $1,200</h3>
            <p>✅ Everything in Basic</p>
            <p>✅ Analytics dashboard</p>
            <p>✅ Payment integration</p>
            <p>✅ 24/7 support</p>
            <button onclick="buyService('pro')">Buy Now</button>
        </div>
        <div class="price-box">
            <h3>Enterprise - $2,500</h3>
            <p>✅ Everything in Pro</p>
            <p>✅ Custom features</p>
            <p>✅ Training included</p>
            <p>✅ 1 year maintenance</p>
            <button onclick="buyService('enterprise')">Buy Now</button>
        </div>
        <br><br>
        <a href="/admin">🔐 Admin Login</a> | 
        <a href="/contact">📞 Contact Us</a>
        <script>
        function buyService(packageName) {
            alert('Redirecting to payment for ' + packageName + ' package!');
            window.location.href = '/buy/' + packageName;
        }
        </script>
    </div>
</body>
</html>
"""

ADMIN_LOGIN = """
<!DOCTYPE html>
<html>
<head>
    <title>Admin Login</title>
    <style>
        body { font-family: Arial; margin: 0; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); height: 100vh; display: flex; align-items: center; justify-content: center; }
        .login-box { background: white; padding: 40px; border-radius: 10px; box-shadow: 0 10px 30px rgba(0,0,0,0.3); }
        input { width: 100%; padding: 10px; margin: 10px 0; border: 1px solid #ddd; border-radius: 5px; }
        button { width: 100%; padding: 12px; background: #007bff; color: white; border: none; border-radius: 5px; cursor: pointer; }
        .error { color: red; margin: 10px 0; }
    </style>
</head>
<body>
    <div class="login-box">
        <h2>🔐 Admin Access</h2>
        {% if error %}
            <div class="error">{{ error }}</div>
        {% endif %}
        <form method="POST">
            <input type="password" name="password" placeholder="Enter Admin Password" required>
            <button type="submit">Login</button>
        </form>
        <p><small>Default password: admin123</small></p>
        <a href="/">← Back to Home</a>
    </div>
</body>
</html>
"""

ADMIN_DASHBOARD = """
<!DOCTYPE html>
<html>
<head>
    <title>Admin Dashboard</title>
    <style>
        body { font-family: Arial; margin: 0; background: #f8f9fa; }
        .header { background: #007bff; color: white; padding: 20px; }
        .container { padding: 20px; }
        .stats { display: flex; gap: 20px; margin: 20px 0; }
        .stat-box { background: white; padding: 20px; border-radius: 10px; flex: 1; text-align: center; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        .customers { background: white; padding: 20px; border-radius: 10px; margin: 20px 0; }
        button { background: #28a745; color: white; padding: 8px 16px; border: none; border-radius: 5px; cursor: pointer; margin: 5px; }
        .logout { background: #dc3545; }
    </style>
</head>
<body>
    <div class="header">
        <h1>💼 Business Dashboard</h1>
        <p>Welcome, Admin! Manage your business here.</p>
    </div>
    <div class="container">
        <div class="stats">
            <div class="stat-box">
                <h3>💰 Total Revenue</h3>
                <h2>${{ revenue }}</h2>
            </div>
            <div class="stat-box">
                <h3>👥 Customers</h3>
                <h2>{{ customer_count }}</h2>
            </div>
            <div class="stat-box">
                <h3>📊 Growth</h3>
                <h2>+25%</h2>
            </div>
        </div>
        <div class="customers">
            <h3>Recent Customers</h3>
            {% if customers %}
                {% for customer in customers %}
                    <p>📧 {{ customer.email }} - ${{ customer.amount }} ({{ customer.package }})</p>
                {% endfor %}
            {% else %}
                <p>No customers yet. Share your link to get started!</p>
            {% endif %}
        </div>
        <button onclick="window.location.href='/'">View Website</button>
        <button onclick="alert('Feature coming soon!')">Send Marketing Email</button>
        <button onclick="alert('Analytics: 45 visitors today!')">Show Analytics</button>
        <button class="logout" onclick="window.location.href='/logout'">Logout</button>
    </div>
</body>
</html>
"""

CONTACT_PAGE = """
<!DOCTYPE html>
<html>
<head>
    <title>Contact Us</title>
    <style>
        body { font-family: Arial; margin: 0; background: #f4f4f4; }
        .container { background: white; padding: 30px; border-radius: 10px; width: 400px; margin: 100px auto; }
        input, textarea { width: 100%; padding: 10px; margin: 10px 0; border: 1px solid #ddd; border-radius: 5px; }
        button { background: #007bff; color: white; padding: 10px 20px; border: none; border-radius: 5px; cursor: pointer; }
    </style>
</head>
<body>
    <div class="container">
        <h2>📞 Contact Us</h2>
        <form method="POST">
            <input type="text" name="name" placeholder="Your Name" required>
            <input type="email" name="email" placeholder="Your Email" required>
            <textarea name="message" placeholder="Message" required></textarea>
            <button type="submit">Send</button>
        </form>
        <a href="/">← Back to Home</a>
    </div>
</body>
</html>
"""

# --- Flask Routes ---
@app.route("/")
def home():
    return render_template_string(HOME_PAGE)

@app.route("/admin", methods=["GET", "POST"])
def admin_login():
    error = None
    if request.method == "POST":
        password = request.form.get("password")
        if password == ADMIN_PASSWORD:
            session["admin"] = True
            return redirect("/dashboard")
        else:
            error = "Incorrect password!"
    return render_template_string(ADMIN_LOGIN, error=error)

@app.route("/dashboard")
def admin_dashboard():
    if not session.get("admin"):
        return redirect("/admin")
    return render_template_string(
        ADMIN_DASHBOARD,
        revenue=REVENUE,
        customer_count=len(CUSTOMERS),
        customers=CUSTOMERS
    )

@app.route("/logout")
def logout():
    session.pop("admin", None)
    return redirect("/")

@app.route("/buy/<package>")
def buy_package(package):
    packages = {"basic": 500, "pro": 1200, "enterprise": 2500}
    if package not in packages:
        flash("Unknown package selected!", "error")
        return redirect("/")
    # In a real app, go to payment page
    return redirect("/contact")

@app.route("/contact", methods=["GET", "POST"])
def contact():
    if request.method == "POST":
        name = request.form.get("name")
        email = request.form.get("email")
        message = request.form.get("message")
        if not (validate_input(name) and validate_input(email) and validate_input(message)):
            flash("All fields must be at least 3 characters.", "error")
        else:
            flash("Message sent! We'll be in touch soon.", "success")
    return render_template_string(CONTACT_PAGE)

@app.route("/register_customer", methods=["POST"])
def register_customer():
    data = request.form
    email = data.get("email")
    amount = data.get("amount")
    package = data.get("package")
    if not (validate_input(email) and validate_input(package)):
        return "Invalid data", 400
    try:
        amt = float(amount)
    except Exception:
        amt = 0
    CUSTOMERS.append({"email": email, "amount": amt, "package": package, "date": datetime.now().strftime("%Y-%m-%d %H:%M")})
    global REVENUE
    REVENUE += amt
    return redirect("/dashboard")

if __name__ == "__main__":
    app.run(debug=True)
