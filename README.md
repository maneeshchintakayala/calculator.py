# calculator.py
import tkinter as tk
from tkinter import ttk
import math

BG_COLOR = "#ffcccc"   # Light red / soft pink
BTN_COLOR = "#ffe6e6"  # Lighter red for buttons

# ---------- Expression Evaluation (O(n)) ----------
def evaluate_expression(expr):
    def precedence(op):
        return {'+':1, '-':1, '*':2, '/':2, '^':3}.get(op, 0)

    def apply_operator(a, b, op):
        if op == '+': return a + b
        if op == '-': return a - b
        if op == '*': return a * b
        if op == '/': return a / b
        if op == '^': return a ** b

    values, ops = [], []
    i = 0

    while i < len(expr):
        if expr[i].isdigit() or expr[i] == '.':
            val = ""
            while i < len(expr) and (expr[i].isdigit() or expr[i] == '.'):
                val += expr[i]
                i += 1
            values.append(float(val))
            continue

        if expr[i] == '(':
            ops.append(expr[i])

        elif expr[i] == ')':
            while ops and ops[-1] != '(':
                b, a = values.pop(), values.pop()
                values.append(apply_operator(a, b, ops.pop()))
            ops.pop()

        elif expr[i] in "+-*/^":
            while ops and precedence(ops[-1]) >= precedence(expr[i]):
                b, a = values.pop(), values.pop()
                values.append(apply_operator(a, b, ops.pop()))
            ops.append(expr[i])

        i += 1

    while ops:
        b, a = values.pop(), values.pop()
        values.append(apply_operator(a, b, ops.pop()))

    return values[0]

# ---------- Scientific Functions ----------
def apply_scientific(func):
    try:
        value = float(entry_var.get())
        result = {
            "sin": math.sin,
            "cos": math.cos,
            "tan": math.tan,
            "sqrt": math.sqrt
        }[func](value)
        history.insert(tk.END, f"{func}({value}) = {result}")
        entry_var.set(str(result))
    except:
        entry_var.set("Error")

# ---------- GUI Functions ----------
def button_click(val):
    entry_var.set(entry_var.get() + val)

def clear():
    entry_var.set("")

def backspace():
    entry_var.set(entry_var.get()[:-1])

def calculate():
    try:
        expr = entry_var.get()
        result = evaluate_expression(expr)
        history.insert(tk.END, f"{expr} = {result}")
        entry_var.set(str(result))
    except:
        entry_var.set("Error")

# ---------- Window ----------
root = tk.Tk()
root.title("Scientific Calculator")
root.geometry("500x550")
root.configure(bg=BG_COLOR)

entry_var = tk.StringVar()
entry = ttk.Entry(root, textvariable=entry_var, font=("Arial", 16))
entry.pack(fill=tk.X, padx=10, pady=10)

# ---------- Buttons ----------
btn_frame = tk.Frame(root, bg=BG_COLOR)
btn_frame.pack()

buttons = [
    '7','8','9','/','⌫',
    '4','5','6','*','(',
    '1','2','3','-',')',
    '0','.','C','=','+',
    '^'
]

row = col = 0
for btn in buttons:
    if btn == 'C':
        action = clear
    elif btn == '⌫':
        action = backspace
    elif btn == '=':
        action = calculate
    else:
        action = lambda x=btn: button_click(x)

    tk.Button(
        btn_frame,
        text=btn,
        width=6,
        height=2,
        bg=BTN_COLOR,
        command=action
    ).grid(row=row, column=col, padx=3, pady=3)

    col += 1
    if col == 5:
        col = 0
        row += 1

# ---------- Scientific Buttons ----------
sci_frame = tk.Frame(root, bg=BG_COLOR)
sci_frame.pack(pady=5)

for f in ["sin", "cos", "tan", "sqrt"]:
    tk.Button(
        sci_frame,
        text=f,
        width=8,
        bg=BTN_COLOR,
        command=lambda x=f: apply_scientific(x)
    ).pack(side=tk.LEFT, padx=5)

# ---------- History ----------
tk.Label(root, text="History", bg=BG_COLOR, font=("Arial", 11, "bold")).pack()
history = tk.Listbox(root, height=7)
history.pack(fill=tk.BOTH, padx=10, pady=5)

root.mainloop()

