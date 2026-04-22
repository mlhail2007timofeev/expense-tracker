# expense-tracker
import tkinter as tk
from tkinter import ttk, messagebox
import json
from datetime import datetime
import os

class ExpenseTracker:
    def __init__(self, root):
        self.root = root
        self.root.title("Expense Tracker - Трекер расходов")
        self.expenses = []
        self.load_data()
        self.create_widgets()

    def create_widgets(self):
        # Поля ввода
        tk.Label(self.root, text="Сумма (руб.):").grid(row=0, column=0, padx=5, pady=5, sticky="w")
        self.amount_entry = tk.Entry(self.root, width=20)
        self.amount_entry.grid(row=0, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Категория:").grid(row=1, column=0, padx=5, pady=5, sticky="w")
        self.category_var = tk.StringVar()
        self.category_combo = ttk.Combobox(
            self.root,
            textvariable=self.category_var,
            values=["Еда", "Транспорт", "Развлечения", "Жильё", "Другое"],
            state="readonly"
        )
        self.category_combo.grid(row=1, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Дата (ГГГГ-ММ-ДД):").grid(row=2, column=0, padx=5, pady=5, sticky="w")
        self.date_entry = tk.Entry(self.root, width=20)
        self.date_entry.grid(row=2, column=1, padx=5, pady=5)

        # Кнопка добавления
        tk.Button(self.root, text="Добавить расход", command=self.add_expense).grid(
            row=3, column=0, columnspan=2, pady=10
        )

        # Таблица
        columns = ("Сумма", "Категория", "Дата")
        self.tree = ttk.Treeview(self.root, columns=columns, show="headings", height=10)
        for col in columns:
            self.tree.heading(col, text=col)
            self.tree.column(col, width=120)
        self.tree.grid(row=4, column=0, columnspan=2, padx=5, pady=5, sticky="nsew")

        # Фильтрация
        tk.Label(self.root, text="Фильтр по категории:").grid(row=5, column=0, padx=5, pady=5, sticky="w")
        self.filter_var = tk.StringVar(value="Все")
        self.filter_combo = ttk.Combobox(
            self.root,
            textvariable=self.filter_var,
            values=["Все", "Еда", "Транспорт", "Развлечения", "Жильё", "Другое"],
            state="readonly"
        )
        self.filter_combo.grid(row=5, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Период с (ГГГГ-ММ-ДД):").grid(row=6, column=0, padx=5, pady=5, sticky="w")
        self.start_date_entry = tk.Entry(self.root, width=20)
        self.start_date_entry.grid(row=6, column=1, padx=5, pady=5)

        tk.Label(self.root, text="по (ГГГГ-ММ-ДД):").grid(row=7, column=0, padx=5, pady=5, sticky="w")
        self.end_date_entry = tk.Entry(self.root, width=20)
        self.end_date_entry.grid(row=7, column=1, padx=5, pady=5)

        tk.Button(self.root, text="Применить фильтр", command=self.apply_filter).grid(
            row=8, column=0, columnspan=2, pady=10
        )

        # Подсчёт суммы
        self.total_label = tk.Label(self.root, text="Общая сумма: 0 руб.", font=("Arial", 12, "bold"))
        self.total_label.grid(row=9, column=0, columnspan=2, pady=10)

    def add_expense(self):
        try:
            # Валидация суммы
            amount_str = self.amount_entry.get().strip()
            if not amount_str:
                raise ValueError("Поле 'Сумма' не может быть пустым")
            amount = float(amount_str)
            if amount <= 0:
                raise ValueError("Сумма должна быть положительным числом")

            # Валидация категории
            category = self.category_var.get()
            if not category:
                raise ValueError("Выберите категорию")

            # Валидация даты
            date_str = self.date_entry.get().strip()
            if not date_str:
                raise ValueError("Поле 'Дата' не может быть пустым")
            try:
                date = datetime.strptime(date_str, "%Y-%m-%d")
            except ValueError:
                raise ValueError("Неверный формат даты. Используйте ГГГГ-ММ-ДД")

            # Добавляем в список
            expense = {"amount": amount, "category": category, "date": date_str}
            self.expenses.append(expense)

            # Обновляем таблицу
            self.tree.insert("", "end", values=(f"{amount:.2f}", category, date_str))

            # Очищаем поля
            self.amount_entry.delete(0, tk.END)
            self.date_entry.delete(0, tk.END)

            # Сохраняем данные
            self.save_data()
            messagebox.showinfo("Успех", "Расход успешно добавлен!")

        except ValueError as e:
            messagebox.showerror("Ошибка ввода", str(e))
        except Exception as e:
            messagebox.showerror("Ошибка", f"Произошла непредвиденная ошибка: {str(e)}")

    def save_data(self):
        with open("expenses.json", "w", encoding="utf-8") as f:
            json.dump(self.expenses, f, ensure_ascii=False, indent=4)

    def load_data(self):
        if os.path.exists("expenses.json"):
            try:
                with open("expenses.json", "r", encoding="utf-8") as f:
                    self.expenses = json.load(f)
                    # Заполняем таблицу при загрузке
                    for expense in self.expenses:
                        self.tree.insert(
                            "", "end",
                            values=(f"{expense['amount']:.2f}", expense["category"], expense["date"])
                )
            except (json.JSONDecodeError, IOError):
                messagebox.showwarning("Предупреждение", "Не удалось загрузить данные из файла expenses.json")
                self.expenses = []
        else:
            self.expenses = []

    def apply_filter(self):
        filtered = self.expenses

        # Фильтр по категории
        category_filter = self.filter_var.get()
        if category_filter != "Все":
            filtered = [e for e in filtered if e["category"] == category_filter]

        # Фильтр по дате
        start_date_str = self.start_date_entry.get().strip()
        end_date_str = self.end_date_entry.get().strip()

        if start_date_str:
            try:
                start_date = datetime.strptime(start_date_str, "%Y-%m-%d")
                filtered = [e for e in filtered
                            if datetime.strptime(e["date"], "%Y-%m-%d") >= start_date]
            except ValueError:
                messagebox.showerror("Ошибка", "Неверный формат начальной даты")
                return

        if end_date_str:
            try:
                end_date = datetime.strptime(end_date_str, "%Y-%m-%d")
                filtered = [e for e in filtered
                          if datetime.strptime(
