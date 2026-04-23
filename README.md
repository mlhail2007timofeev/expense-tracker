import tkinter as tk
from tkinter import ttk, messagebox
import requests
import json
import os

class CurrencyConverter:
    def __init__(self, root, api_key):
        self.root = root
        self.root.title("Currency Converter - Конвертер валют")
        self.api_key = api_key
        self.history = []
        self.currencies = self.load_currencies()
        self.create_widgets()
        self.load_history()

    def load_currencies(self):
        """Загружаем список валют (можно расширить)"""
        return ["USD", "EUR", "GBP", "JPY", "CNY", "RUB"]

    def create_widgets(self):
        # Выбор валют
        input_frame = tk.Frame(self.root)
        input_frame.pack(pady=10, padx=10, fill="x")

        tk.Label(input_frame, text="Из валюты:").grid(row=0, column=0, padx=5, pady=5, sticky="w")
        self.from_currency = ttk.Combobox(input_frame, values=self.currencies, width=15)
        self.from_currency.set("USD")
        self.from_currency.grid(row=0, column=1, padx=5, pady=5)

        tk.Label(input_frame, text="В валюту:").grid(row=1, column=0, padx=5, pady=5, sticky="w")
        self.to_currency = ttk.Combobox(input_frame, values=self.currencies, width=15)
        self.to_currency.set("EUR")
        self.to_currency.grid(row=1, column=1, padx=5, pady=5)

        # Поле ввода суммы
        tk.Label(input_frame, text="Сумма:").grid(row=2, column=0, padx=5, pady=5, sticky="w")
        self.amount_entry = tk.Entry(input_frame, width=20)
        self.amount_entry.grid(row=2, column=1, padx=5, pady=5)

        # Кнопка конвертации
        tk.Button(input_frame, text="Конвертировать",
               command=self.convert_currency).grid(row=3, column=0, columnspan=2, pady=10)

        # Результат
        self.result_label = tk.Label(input_frame, text="", font=("Arial", 12))
        self.result_label.grid(row=4, column=0, columnspan=2, pady=5)

        # История конвертаций
        tk.Label(self.root, text="История конвертаций:",
               font=("Arial", 12, "bold")).pack(anchor="w", padx=10, pady=(10, 5))

        columns = ("Сумма", "Из", "В", "Результат")
        self.history_tree = ttk.Treeview(self.root, columns=columns, show="headings", height=10)
        for col in columns:
            self.history_tree.heading(col, text=col)
            self.history_tree.column(col, width=120)
        self.history_tree.pack(pady=5, padx=10, fill="both", expand=True)

    def convert_currency(self):
        try:
            # Валидация суммы
            amount_str = self.amount_entry.get().strip()
            if not amount_str:
                raise ValueError("Поле 'Сумма' не может быть пустым")
            amount = float(amount_str)
            if amount <= 0:
                raise ValueError("Сумма должна быть положительным числом")

            from_curr = self.from_currency.get()
            to_curr = self.to_currency.get()

            if from_curr == to_curr:
                result = amount
            else:
                # Получаем курс через API
                rate = self.get_exchange_rate(from_curr, to_curr)
                result = amount * rate

            # Отображаем результат
            result_text = f"{amount} {from_curr} = {result:.2f} {to_curr}"
            self.result_label.config(text=result_text)

            # Добавляем в историю
            record = {
                "amount": amount,
                "from": from_curr,
                "to": to_curr,
                "result": result
            }
            self.history.append(record)
            self.update_history_display()
            self.save_history()

        except ValueError as e:
            messagebox.showerror("Ошибка ввода", str(e))
        except Exception as e:
            messagebox.showerror("Ошибка API", f"Не удалось получить курс: {str(e)}")

    def get_exchange_rate(self, from_curr, to_curr):
        """Получаем курс через внешний API"""
        url = f"https://api.exchangerate-api.com/v4/latest/{from_curr}"
        response = requests.get(url)
        data = response.json()
        if to_curr in data["rates"]:
            return data["rates"][to_curr]
        else:
            raise Exception(f"Валюта {to_curr} не найдена в API")

    def update_history_display(self):
        for item in self.history_tree.get_children():
            self.history_tree.delete(item)
        for record in reversed(self.history[-20:]):  # Последние 20 записей
            self.history_tree.insert("", "end", values=(
                f"{record['amount']:.2f}",
                record["from"],
                record["to"],
                f"{record['result']:.2f}"
            ))

    def save_history(self):
        with open("conversion_history.json", "w", encoding="utf-8") as f:
            json.dump(self.history, f, ensure_ascii=False, indent=4)


    def load_history(self):
        if os.path.exists("conversion_history.json"):
            try:
                with open("conversion_history.json", "r", encoding="utf-8") as f:
                    self.history = json.load(f)
                self.update_history_display()
            except (json.JSONDecodeError, IOError) as e:
                print(f"Предупреждение: не удалось загрузить историю: {e}")
        else:
            self.history = []

def main():
    # Замените на ваш реальный API-ключ
    API_KEY = "your_api_key_here"

    root = tk.Tk()
    app = CurrencyConverter(root, API_KEY)
    root.mainloop()

if __name__ == "__main__":
    main()
