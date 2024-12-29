# Banking-System
import sqlite3
import random
import re
from datetime import datetime

# Initialize database
DB_NAME = 'banking_system.db'

# Utility Functions

def validate_email(email):
    pattern = r'^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$'
    return re.match(pattern, email)

def validate_contact(contact):
    return contact.isdigit() and len(contact) == 10

def validate_password(password):
    pattern = r'^(?=.*[A-Za-z])(?=.*\d)(?=.*[@$!%*#?&])[A-Za-z\d@$!%*#?&]{8,}$'
    return re.match(pattern, password)

def generate_account_number():
    return ''.join([str(random.randint(0, 9)) for _ in range(10)])

def create_tables():
    conn = sqlite3.connect(DB_NAME)
    cursor = conn.cursor()

    cursor.execute('''CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        account_number TEXT UNIQUE NOT NULL,
        dob TEXT NOT NULL,
        city TEXT NOT NULL,
        password TEXT NOT NULL,
        balance REAL NOT NULL,
        contact_number TEXT NOT NULL,
        email TEXT UNIQUE NOT NULL,
        address TEXT NOT NULL,
        is_active INTEGER DEFAULT 1
    )''')

    cursor.execute('''CREATE TABLE IF NOT EXISTS transactions (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        account_number TEXT NOT NULL,
        type TEXT NOT NULL,
        amount REAL NOT NULL,
        timestamp TEXT NOT NULL,
        FOREIGN KEY (account_number) REFERENCES users(account_number)
    )''')

    conn.commit()
    conn.close()

def add_user():
    name = input("Enter Name: ").strip()
    dob = input("Enter DOB (YYYY-MM-DD): ").strip()
    city = input("Enter City: ").strip()
    password = input("Enter Password: ").strip()
    initial_balance = float(input("Enter Initial Balance (minimum 2000): ").strip())
    contact_number = input("Enter Contact Number: ").strip()
    email = input("Enter Email: ").strip()
    address = input("Enter Address: ").strip()

    if len(name) == 0 or not validate_password(password):
        print("Invalid Name or Password!")
        return

    if not validate_contact(contact_number):
        print("Invalid Contact Number!")
        return

    if not validate_email(email):
        print("Invalid Email!")
        return

    if initial_balance < 2000:
        print("Initial balance must be at least 2000!")
        return

    account_number = generate_account_number()
    dob_format = datetime.strptime(dob, '%Y-%m-%d').strftime('%Y-%m-%d')

    conn = sqlite3.connect(DB_NAME)
    cursor = conn.cursor()
    cursor.execute('''INSERT INTO users (name, account_number, dob, city, password, balance, contact_number, email, address)
                      VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)''',
                   (name, account_number, dob_format, city, password, initial_balance, contact_number, email, address))
    conn.commit()
    conn.close()

    print(f"User created successfully! Account Number: {account_number}")

def show_users():
    conn = sqlite3.connect(DB_NAME)
    cursor = conn.cursor()
    cursor.execute("SELECT name, account_number, dob, city, balance, contact_number, email, address, is_active FROM users")

    users = cursor.fetchall()
    if not users:
        print("No users found!")
    else:
        for user in users:
            print(f"Name: {user[0]}, Account Number: {user[1]}, DOB: {user[2]}, City: {user[3]}, Balance: {user[4]}, Contact: {user[5]}, Email: {user[6]}, Address: {user[7]}, Active: {'Yes' if user[8] else 'No'}")

    conn.close()

def login():
    account_number = input("Enter Account Number: ").strip()
    password = input("Enter Password: ").strip()

    conn = sqlite3.connect(DB_NAME)
    cursor = conn.cursor()
    cursor.execute("SELECT password, is_active FROM users WHERE account_number = ?", (account_number,))
    result = cursor.fetchone()

    if result and result[0] == password and result[1] == 1:
        print("Login successful!")
        user_dashboard(account_number)
    else:
        print("Invalid credentials or account is inactive!")

    conn.close()

def user_dashboard(account_number):
    while True:
        print("\n1. Show Balance")
        print("2. Show Transactions")
        print("3. Credit Amount")
        print("4. Debit Amount")
        print("5. Transfer Amount")
        print("6. Activate/Deactivate Account")
        print("7. Change Password")
        print("8. Update Profile")
        print("9. Logout")

        choice = input("Enter your choice: ")

        if choice == '1':
            show_balance(account_number)
        elif choice == '2':
            show_transactions(account_number)
        elif choice == '3':
            credit_amount(account_number)
        elif choice == '4':
            debit_amount(account_number)
        elif choice == '5':
            transfer_amount(account_number)
        elif choice == '6':
            toggle_account_status(account_number)
        elif choice == '7':
            change_password(account_number)
        elif choice == '8':
            update_profile(account_number)
        elif choice == '9':
            print("Logged out successfully!")
            break
        else:
            print("Invalid choice! Please try again.")

def main():
    create_tables()

    while True:
        print("\n--- Banking System ---")
        print("1. Add User")
        print("2. Show Users")
        print("3. Login")
        print("4. Exit")

        choice = input("Enter your choice: ")

        if choice == '1':
            add_user()
        elif choice == '2':
            show_users()
        elif choice == '3':
            login()
        elif choice == '4':
            print("Exiting... Goodbye!")
            break
        else:
            print("Invalid choice! Please try again.")

if __name__ == "__main__":
    main()
