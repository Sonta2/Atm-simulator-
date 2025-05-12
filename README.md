# Atm-simulator-
Atm simulator 
import tkinter as tk
import ttkbootstrap as tb
from ttkbootstrap import Style
from tkinter import ttk
from tkinter import messagebox, simpledialog
import datetime
import json
import os
import matplotlib.pyplot as plt
import hashlib

class ATMSimulator:
    def __init__(self, root):
        self.root = root
        self.style = Style(theme='flatly')
        self.root.title("ATM Simulator")
        self.root.geometry("400x650")
        self.root.configure(bg="#f0f0f0")

        self.balance = 1000
        self.transaction_history = []
        self.accounts = {}
        self.current_account = None
        self.budget = 0

        self.create_widgets()
        self.load_accounts()

    def create_widgets(self):
        self.title_label = tk.Label(self.root, text="ATM Simulator", font=("Helvetica", 18, "bold"), bg="#f0f0f0")
        self.title_label.pack(pady=20)

        self.login_frame = ttk.Frame(self.root)
        self.login_frame.pack(pady=20)
        ttk.Label(self.login_frame, text="Username:").grid(row=0, column=0)
        self.username_entry = ttk.Entry(self.login_frame)
        self.username_entry.grid(row=0, column=1)

        ttk.Label(self.login_frame, text="Password:").grid(row=1, column=0)
        self.password_entry = ttk.Entry(self.login_frame, show="*")
        self.password_entry.grid(row=1, column=1)

        ttk.Button(self.login_frame, text="Login / Register", command=self.login).grid(row=2, columnspan=2, pady=5)

        self.user_label = tk.Label(self.root, text="Not logged in", bg="#f0f0f0", font=("Helvetica", 10, "italic"))
        self.user_label.pack()

        self.balance_label = tk.Label(self.root, text=f"Current Balance: ${self.balance:.2f}", font=("Helvetica", 14), bg="#f0f0f0")
        self.balance_label.pack(pady=10)

        self.buttons = []
        for text, command in [
            ("Check Balance", self.check_balance),
            ("Deposit", self.deposit),
            ("Withdraw", self.withdraw),
            ("Transaction History", self.view_history),
            ("Set Budget", self.set_budget),
            ("Visualize Spending", self.visualize_spending),
            ("Logout", self.logout),
            ("Exit", self.root.quit),
        ]:
            btn = ttk.Button(self.root, text=text, command=command)
            btn.pack(pady=5)
            self.buttons.append(btn)

        self.toggle_buttons(state="disabled")

    def toggle_buttons(self, state="normal"):
        for btn in self.buttons:
            btn.config(state=state)

    def load_accounts(self):
        if os.path.exists('accounts.json'):
            with open('accounts.json', 'r') as file:
                self.accounts = json.load(file)

    def save_accounts(self):
        with open('accounts.json', 'w') as file:
            json.dump(self.accounts, file)

    def hash_password(self, password):
        return hashlib.sha256(password.encode()).hexdigest()

    def login(self):
        username = self.username_entry.get()
        password = self.password_entry.get()
        if not username or not password:
            messagebox.showerror("Error", "Please enter both username and password.")
            return
        hashed_pw = self.hash_password(password)

        if username in self.accounts:
            if self.accounts[username]['password'] != hashed_pw:
                messagebox.showerror("Login Failed", "Incorrect password.")
                return
            else:
                messagebox.showinfo("Login Success", f"Welcome back, {username}!")
        else:
            self.accounts[username] = {
                "password": hashed_pw,
                "balance": 1000,
                "transactions": [],
                "budget": 0
            }
            messagebox.showinfo("Account Created", "New account registered!")

        self.current_account = username
        self.balance = self.accounts[username]['balance']
        self.budget = self.accounts[username].get('budget', 0)
        self.balance_label.config(text=f"Current Balance: ${self.balance:.2f}")
        self.user_label.config(text=f"Logged in as: {username}")
        self.toggle_buttons("normal")
        self.save_accounts()

    def logout(self):
        self.current_account = None
        self.balance = 1000
        self.budget = 0
        self.balance_label.config(text=f"Current Balance: ${self.balance:.2f}")
        self.user_label.config(text="Not logged in")
        self.username_entry.delete(0, tk.END)
        self.password_entry.delete(0, tk.END)
        self.toggle_buttons("disabled")
        messagebox.showinfo("Logout", "You have been logged out.")

    def check_balance(self):
        messagebox.showinfo("Balance", f"Your balance is: ${self.balance:.2f}")

    def deposit(self):
        amount = simpledialog.askfloat("Deposit", "Enter amount to deposit:")
        if amount and amount > 0:
            self.balance += amount
            self.record_transaction('Deposit', amount)
            self.accounts[self.current_account]['balance'] = self.balance
            self.balance_label.config(text=f"Current Balance: ${self.balance:.2f}")
            self.animate_label(self.balance_label, '#d4f4dd')
            messagebox.showinfo("Deposit Successful", f"Deposited: ${amount:.2f}")
            self.save_accounts()

    def withdraw(self):
        amount = simpledialog.askfloat("Withdraw", "Enter amount to withdraw:")
        if amount:
            if amount > self.balance:
                messagebox.showerror("Insufficient Funds", "You do not have enough funds.")
            elif amount <= 0:
                messagebox.showerror("Invalid Amount", "Enter a positive amount.")
            else:
                self.balance -= amount
                self.record_transaction('Withdraw', amount)
                self.accounts[self.current_account]['balance'] = self.balance
                self.balance_label.config(text=f"Current Balance: ${self.balance:.2f}")
                self.animate_label(self.balance_label, '#f4d4d4')
                messagebox.showinfo("Withdrawal Successful", f"Withdrew: ${amount:.2f}")
                self.save_accounts()

    def record_transaction(self, t_type, amount):
        transaction = {
            'date': datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            'type': t_type,
            'amount': amount
        }
        self.accounts[self.current_account]['transactions'].append(transaction)

    def view_history(self):
        transactions = self.accounts[self.current_account]['transactions']
        if not transactions:
            messagebox.showinfo("Transaction History", "No transactions made yet!")
        else:
            history_message = "Transaction History:\n"
            for t in transactions:
                history_message += f"{t['date']} - {t['type']} - ${t['amount']:.2f}\n"
            messagebox.showinfo("Transaction History", history_message)

    def set_budget(self):
        budget = simpledialog.askfloat("Set Budget", "Enter your monthly budget:")
        if budget and budget > 0:
            self.budget = budget
            self.accounts[self.current_account]['budget'] = budget
            messagebox.showinfo("Budget Set", f"Budget set to: ${budget:.2f}")
            self.save_accounts()
        else:
            messagebox.showerror("Invalid Amount", "Please enter a positive budget.")

    def visualize_spending(self):
        transactions = self.accounts[self.current_account]['transactions']
        total_spent = sum(t['amount'] for t in transactions if t['type'] == 'Withdraw')
        categories = ['Spent', 'Remaining']
        values = [total_spent, max(0, self.budget - total_spent)]

        plt.figure(figsize=(7, 5))
        plt.bar(categories, values, color=['red', 'green'])
        plt.title('Spending vs. Budget')
        plt.xlabel('Categories')
        plt.ylabel('Amount ($)')
        plt.ylim(0, max(self.budget, total_spent) + 100)
        plt.axhline(y=self.budget, color='blue', linestyle='--', label='Budget Line')
        plt.legend()
        plt.show()

    def animate_label(self, label, flash_color, steps=6, interval=100):
        orig_bg = label.cget('background')
        def flash(count):
            label.config(background=flash_color if count % 2 == 0 else orig_bg)
            if count < steps:
                self.root.after(interval, lambda: flash(count+1))
        flash(0)

if __name__ == "__main__":
    root = tb.Window(themename='flatly')
    app = ATMSimulator(root)
    root.mainloop()
