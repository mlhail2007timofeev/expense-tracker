# expense-tracker
import tkinter as tk
from tkinter import ttk, messagebox
import json
from datetime import datetime
import os

class WeatherDiary:
    def __init__(self, root):
        self.root = root
        self.root.title("Weather Diary - Дневник погоды")
        self.records = []
        self.create_widgets()
        # Загружаем данные после создания всех виджетов
        self.load_data()

    def create_widgets(self):
        # Форма ввода
        input_frame = tk.Frame(self.root)
        input_frame.pack(pady=10, padx=10, fill="x")

        # Дата
        tk.Label(input_frame, text="Дата (ГГГГ-ММ-ДД):").grid(row=0, column=0, padx=5, pady=5, sticky="w")
        self.date_entry = tk.Entry(input_frame, width=20)
        self.date_entry.grid(row=0, column=1, padx=5, pady=5)

        # Температура
        tk.Label(input_frame, text="Температура (°C):").grid(row=1, column=0, padx=5, pady=5, sticky="w")
        self.temp_entry = tk.Entry(input_frame, width=20)
        self.temp_entry.grid(row=1, column=1, padx=5, pady=5)

        # Описание погоды
        tk.Label(input_frame, text="Описание погоды:").grid(row=2, column=0, padx=5, pady=5, sticky="w")
        self.desc_entry = tk.Entry(input_frame, width=20)
        self.desc_entry.grid(row=2, column=1, padx=5, pady=5)

        # Осадки
        tk.Label(input_frame, text="Осадки:").grid(row=3, column=0, padx=5, pady=5, sticky="w")
        self.rain_var = tk.StringVar(value="Нет")
        tk.Radiobutton(input_frame, text="Да", variable=self.rain_var, value="Да").grid(row=3, column=1, sticky="w")
        tk.Radiobutton(input_frame, text="Нет", variable=self.rain_var, value="Нет").grid(row=3, column=2, sticky="w")

        # Кнопка добавления
        tk.Button(input_frame, text="Добавить запись", command=self.add_record).grid(
            row=4, column=0, columnspan=3, pady=10
        )

        # Таблица записей
        table_frame = tk.Frame(self.root)
        table_frame.pack(pady=10, padx=10, fill="both", expand=True)

        columns = ("Дата", "Температура (°C)", "Описание", "Осадки")
        self.tree = ttk.Treeview(table_frame, columns=columns, show="headings", height=10)
        for col in columns:
            self.tree.heading(col, text=col)
            self.tree.column(col, width=120)
        self.tree.pack(fill="both", expand=True)

        # Фильтрация
        filter_frame = tk.Frame(self.root)
        filter_frame.pack(pady=10, padx=10, fill="x")

        tk.Label(filter_frame, text="Фильтр по дате:").grid(row=0, column=0, padx=5, pady=5, sticky="w")
        self.filter_date = tk.Entry(filter_frame, width=20)
        self.filter_date.grid(row=0, column=1, padx=5, pady=5)

        tk.Label(filter_frame, text="Температура выше (°C):").grid(row=1, column=0, padx=5, pady=5, sticky="w")
        self.filter_temp = tk.Entry(filter_frame, width=20)
        self.filter_temp.grid(row=1, column=1, padx=5, pady=5)

        tk.Button(filter_frame, text="Применить фильтр", command=self.apply_filter).grid(row=2, column=0, pady=5)
        tk.Button(filter_frame, text="Сбросить фильтр", command=self.reset_filter).grid(row=2, column=1, pady=5)

    def add_record(self):
        try:
            # Валидация даты
            date_str = self.date_entry.get().strip()
            if not date_str:
                raise ValueError("Поле 'Дата' не может быть пустым")
            try:
                date = datetime.strptime(date_str, "%Y-%m-%d")
            except ValueError:
                raise ValueError("Неверный формат даты. Используйте ГГГГ-ММ-ДД")

            # Валидация температуры
            temp_str = self.temp_entry.get().strip()
            if not temp_str:
                raise ValueError("Поле 'Температура' не может быть пустым")
            temperature = float(temp_str)

            # Валидация описания
            description = self.desc_entry.get().strip()
            if not description:
                raise ValueError("Поле 'Описание' не может быть пустым")

            # Осадки
            rain = self.rain_var.get()

            # Добавляем запись
            record = {
                "date": date_str,
                "temperature": temperature,
                "description": description,
                "rain": rain
            }
            self.records.append(record)

            # Обновляем таблицу
            self.tree.insert("", "end", values=(
                date_str, f"{temperature}°C", description, rain
            ))

            # Очищаем поля
            self.date_entry.delete(0, tk.END)
            self.temp_entry.delete(0, tk.END)
            self.desc_entry.delete(0, tk.END)
            self.rain_var.set("Нет")

            # Сохраняем данные
            self.save_data()
            messagebox.showinfo("Успех", "Запись о погоде успешно добавлена!")
        except ValueError as e:
            messagebox.showerror("Ошибка ввода", str(e))
        except Exception as e:
            messagebox.showerror("Ошибка", f"Произошла непредвиденная ошибка: {str(e)}")

    def save_data(self):
        with open("weather_records.json", "w", encoding="utf-8") as f:
            json.dump(self.records, f, ensure_ascii=False, indent=4)

    def load_data(self):
        if os.path.exists("weather_records.json"):
            try:
                with open("weather_records.json", "r", encoding="utf-8") as f:
                    self.records = json.load(f)
                # Заполняем таблицу при загрузке
                for record in self.records:
                    self.tree.insert(
                "", "end",
                values=(
                    record["date"],
            f"{record['temperature']}°C",
            record["description"],
            record["rain"]
        ))
            except (json.JSONDecodeError, IOError) as e:
                print(f"Предупреждение: не удалось загрузить данные: {e}")
        else:
            self.records = []

    def apply_filter(self):
        filtered = self.records

        # Фильтр по дате
        filter_date = self.filter_date.get().strip()
        if filter_date:
            filtered = [r for r in filtered if r["date"] == filter_date]

        # Фильтр по температуре
        filter_temp_str = self.filter_temp.get().strip()
        if filter_temp_str:
            try:
                filter_temp = float(filter_temp_str)
                filtered = [r for r in filtered if r["temperature"] >= filter_temp]
            except ValueError:
                messagebox.showwarning("Предупреждение", "Неверный формат температуры в фильтре")
                return

        # Обновляем таблицу
        for item in self.tree.get_children():
            self.tree.delete(item)
        for record in filtered:
            self.tree.insert("", "end", values=(
                record["date"], f"{record['temperature']}°C", record["description"], record
