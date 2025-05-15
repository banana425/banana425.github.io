from flask import Flask, render_template, request, redirect, url_for
import sqlite3
from sqlite3 import Error

app = Flask(__name__)

def init_db():
    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS users
                 (id INTEGER PRIMARY KEY AUTOINCREMENT,
                  name TEXT NOT NULL,
                  address TEXT NOT NULL,
                  phone TEXT)''')
    c.execute('''CREATE TABLE IF NOT EXISTS notes
                 (id INTEGER PRIMARY KEY AUTOINCREMENT,
                  user_id INTEGER,
                  content TEXT NOT NULL,
                  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                  FOREIGN KEY (user_id) REFERENCES users (id))''')
    conn.commit()
    conn.close()

@app.route('/user/<int:user_id>')
def user_details(user_id):
    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    c.execute('SELECT * FROM users WHERE id = ?', (user_id,))
    user = c.fetchone()
    c.execute('SELECT * FROM notes WHERE user_id = ? ORDER BY created_at DESC', (user_id,))
    notes = c.fetchall()
    conn.close()
    return render_template('user_details.html', user=user, notes=notes)

@app.route('/add_note/<int:user_id>', methods=['POST'])
def add_note(user_id):
    content = request.form['note_content']
    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    c.execute('INSERT INTO notes (user_id, content) VALUES (?, ?)',
              (user_id, content))
    conn.commit()
    conn.close()
    return redirect(url_for('user_details', user_id=user_id))

@app.route('/')
def index():
    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    c.execute('SELECT * FROM users')
    users = c.fetchall()
    conn.close()
    return render_template('index.html', users=users)

@app.route('/add_user', methods=['POST'])
def add_user():
    name = request.form['name']
    address = request.form['address']
    phone = request.form['phone']

    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    c.execute('INSERT INTO users (name, address, phone) VALUES (?, ?, ?)',
              (name, address, phone))
    conn.commit()
    conn.close()
    return redirect(url_for('index'))

@app.route('/search')
def search():
    query = request.args.get('query', '')
    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    c.execute('''SELECT * FROM users 
                 WHERE name LIKE ? OR address LIKE ? OR phone LIKE ?''',
              (f'%{query}%', f'%{query}%', f'%{query}%'))
    users = c.fetchall()
    conn.close()
    return render_template('index.html', users=users, search_query=query)

@app.route('/edit_user', methods=['POST'])
def edit_user():
    id = request.form['id']
    name = request.form['name']
    email = request.form['email']
    phone = request.form['phone']

    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    c.execute('UPDATE users SET name=?, email=?, phone=? WHERE id=?',
              (name, email, phone, id))
    conn.commit()
    conn.close()
    return redirect(url_for('index'))

@app.route('/delete_user/<int:id>', methods=['POST'])
def delete_user(id):
    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    c.execute('DELETE FROM users WHERE id=?', (id,))
    conn.commit()
    conn.close()
    return redirect(url_for('index'))

def add_test_data():
    conn = sqlite3.connect('users.db')
    c = conn.cursor()
    # Clear existing data
    c.execute('DELETE FROM users')
    c.execute('DELETE FROM notes')
    conn.commit()
    conn.close()

if __name__ == '__main__':
    init_db()
    add_test_data()  # Adding test data
    app.run(host='0.0.0.0', port=5000)
