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
        self.create_widgets()
        self.load_data()

    def create_widgets(self):
        # Форма ввода
        input_frame = tk.Frame(self.root)
        input_frame.pack(pady=10, padx=10, fill="x")

        # Сумма
        tk.Label(input_frame, text="Сумма:").grid(row=0, column=0, padx=5, pady=5, sticky="w")
        self.amount_entry = tk.Entry(input_frame, width=20)
        self.amount_entry.grid(row=0, column=1, padx=5, pady=5)

        # Категория
        tk.Label(input_frame, text="Категория:").grid(row=1, column=0, padx=5, pady=5, sticky="w")
        self.category_entry = ttk.Combobox(input_frame, values=[
            "Еда", "Транспорт", "Развлечения", "Жильё", "Одежда", "Другое"
        ], width=17)
        self.category_entry.grid(row=1, column=1, padx=5, pady=5)

        # Дата
        tk.Label(input_frame, text="Дата (ГГГГ-ММ-ДД):").grid(row=2, column=0, padx=5, pady=5, sticky="w")
        self.date_entry = tk.Entry(input_frame, width=20)
        self.date_entry.grid(row=2, column=1, padx=5, pady=5)

        # Кнопка добавления
        tk.Button(input_frame, text="Добавить расход", command=self.add_expense).grid(
            row=3, column=0, columnspan=2, pady=10
        )

        # Таблица расходов
        table_frame = tk.Frame(self.root)
        table_frame.pack(pady=10, padx=10, fill="both", expand=True)

        columns = ("Дата", "Сумма", "Категория")
        self.tree = ttk.Treeview(table_frame, columns=columns, show="headings", height=10)
        for col in columns:
            self.tree.heading(col, text=col)
            self.tree.column(col, width=150)
        self.tree.pack(fill="both", expand=True)

        # Фильтрация и подсчёт суммы
        filter_frame = tk.Frame(self.root)
        filter_frame.pack(pady=10, padx=10, fill="x")

        tk.Label(filter_frame, text="Фильтр по категории:").grid(row=0, column=0, padx=5, pady=5, sticky="w")
        self.filter_category = ttk.Combobox(filter_frame, values=["Все"] + [
            "Еда", "Транспорт", "Развлечения", "Жильё", "Одежда", "Другое"
        ])
        self.filter_category.set("Все")
        self.filter_category.grid(row=0, column=1, padx=5, pady=5)

        tk.Label(filter_frame, text="Фильтр по дате:").grid(row=1, column=0, padx=5, pady=5, sticky="w")
        self.filter_date = tk.Entry(filter_frame, width=20)
        self.filter_date.grid(row=1, column=1, padx=5, pady=5)

        tk.Button(filter_frame, text="Применить фильтр", command=self.apply_filter).grid(row=2, column=0, pady=5)
        tk.Button(filter_frame, text="Сбросить фильтр", command=self.reset_filter).grid(row=2, column=1, pady=5)

        # Подсчёт суммы за период
        sum_frame = tk.Frame(self.root)
        sum_frame.pack(pady=5, padx=10, fill="x")

        tk.Label(sum_frame, text="Начальная дата (ГГГГ-ММ-ДД):").grid(row=0, column=0, padx=5, pady=5, sticky="w")
        self.start_date_entry = tk.Entry(sum_frame, width=20)
        self.start_date_entry.grid(row=0, column=1, padx=5, pady=5)

        tk.Label(sum_frame, text="Конечная дата (ГГГГ-ММ-ДД):").grid(row=1, column=0, padx=5, pady=5, sticky="w")
        self.end_date_entry = tk.Entry(sum_frame, width=20)
        self.end_date_entry.grid(row=1, column=1, padx=5, pady=5)

        tk.Button(sum_frame, text="Посчитать сумму за период", command=self.calculate_sum).grid(row=2, column=0, columnspan=2, pady=5)

        self.sum_label = tk.Label(sum_frame, text="Общая сумма за период: 0 руб.")
        self.sum_label.grid(row=3, column=0, columnspan=2, pady=5)

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
            category = self.category_entry.get().strip()
            if not category:
                raise ValueError("Выберите категорию расхода")

            # Валидация даты
            date_str = self.date_entry.get().strip()
            if not date_str:
                raise ValueError("Поле 'Дата' не может быть пустым")
            try:
                date = datetime.strptime(date_str, "%Y-%m-%d")
            except ValueError:
                raise ValueError("Неверный формат даты. Используйте ГГГГ-ММ-ДД")

            # Добавляем запись
            expense = {
                "date": date_str,
                "amount": amount,
                "category": category
            }
            self.expenses.append(expense)

            # Обновляем таблицу
            self.tree.insert("", "end", values=(
                date_str, f"{amount} руб.", category
            ))

            # Очищаем поля
            self.amount_entry.delete(0, tk.END)
            self.category_entry.set("")
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
                ("", "end",
                values=(
                    expense["date"],
            f"{expense['amount']} руб.",
            expense["category"]
        ))
            except (json.JSONDecodeError, IOError) as e:
                print(f"Предупреждение: не удалось загрузить данные: {e}")
                self
