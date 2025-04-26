# testrepo

from flask import Flask, render_template, request, redirect, url_for, session
import os
import csv

app = Flask(__name__)
app.secret_key = os.urandom(24)

CSV_FILE = 'impact_data.csv'  # Define CSV file name

# Route for the home page (WeOne Dashboard)
@app.route('/')
def dashboard():
    impacts = []
    if os.path.exists(CSV_FILE):
        with open(CSV_FILE, newline='') as csvfile:
            reader = csv.DictReader(csvfile)
            impacts = list(reader)
    return render_template('index.html', impact_data=impacts)

# Route to handle impact report submission
@app.route('/save-impact', methods=['POST'])
def save_impact():
    date_used = request.form.get('date_used')
    location = request.form.get('location')
    beneficiary = request.form.get('beneficiary')
    purpose = request.form.get('purpose')

    fieldnames = ['date_used', 'location', 'beneficiary', 'purpose']
    write_header = not os.path.exists(CSV_FILE)

    with open(CSV_FILE, 'a', newline='') as csvfile:
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
        if write_header:
            writer.writeheader()
        writer.writerow({
            'date_used': date_used,
            'location': location,
            'beneficiary': beneficiary,
            'purpose': purpose
        })

    return redirect(url_for('dashboard'))

# Admin login route
@app.route('/admin-login', methods=['GET', 'POST'])
def admin_login():
    if request.method == 'POST':
        password = request.form.get('admin-password')
        if password == "admin123":
            session['admin_logged_in'] = True
            return redirect(url_for('admin_dashboard'))
        else:
            return "Incorrect password. Please try again."
    return render_template('admin_login.html')

# Admin dashboard route
@app.route('/admin-dashboard')
def admin_dashboard():
    # Strict check: only allow if session has 'admin_logged_in' and it's True
    if session.get('admin_logged_in') != True:
        return redirect(url_for('admin_login'))
    return render_template('admin_dashboard.html')

# Logout route
@app.route('/logout')
def logout():
    session.pop('admin_logged_in', None)
    return redirect(url_for('admin_login'))

if __name__ == "__main__":
    app.run(debug=True)
