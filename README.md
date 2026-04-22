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
        self.weather_records = []
        self.load_data()
        self.create_widgets()

    def create_widgets(self):
        # Поля ввода
        tk.Label(self.root, text="Дата (ГГГГ-ММ-ДД):").grid(row=0, column=0, padx=5, pady=5, sticky="w")
        self.date_entry = tk.Entry(self.root, width=20)
        self.date_entry.grid(row=0, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Температура (°C):").grid(row=1, column=0, padx=5, pady=5, sticky="w")
        self.temp_entry = tk.Entry(self.root, width=20)
        self.temp_entry.grid(row=1, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Описание погоды:").grid(row=2, column=0, padx=5, pady=5, sticky="w")
        self.desc_entry = tk.Entry(self.root, width=40)
        self.desc_entry.grid(row=2, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Осадки:").grid(row=3, column=0, padx=5, pady=5, sticky="w")
        self.precip_var = tk.StringVar(value="Нет")
        tk.Radiobutton(self.root, text="Да", variable=self.precip_var, value="Да").grid(row=3, column=1, sticky="w")
        tk.Radiobutton(self.root, text="Нет", variable=self.precip_var, value="Нет").grid(row=3, column=1, sticky="e")

        # Кнопка добавления
        tk.Button(self.root, text="Добавить запись", command=self.add_record).grid(
            row=4, column=0, columnspan=2, pady=10
        )

        # Таблица
        columns = ("Дата", "Температура", "Описание", "Осадки")
        self.tree = ttk.Treeview(self.root, columns=columns, show="headings", height=10)
        for col in columns:
            self.tree.heading(col, text=col)
            self.tree.column(col, width=120)
        self.tree.grid(row=5, column=0, columnspan=2, padx=5, pady=5, sticky="nsew")

        # Фильтрация
        tk.Label(self.root, text="Фильтр по дате:").grid(row=6, column=0, padx=5, pady=5, sticky="w")
        self.filter_date_entry = tk.Entry(self.root, width=20)
        self.filter_date_entry.grid(row=6, column=1, padx=5, pady=5)

        tk.Label(self.root, text="Температура выше (°C):").grid(row=7, column=0, padx=5, pady=5, sticky="w")
        self.filter_temp_entry = tk.Entry(self.root, width=20)
        self.filter_temp_entry.grid(row=7, column=1, padx=5, pady=5)

        tk.Button(self.root, text="Применить фильтр", command=self.apply_filter).grid(
            row=8, column=0, columnspan=2, pady=10
        )

        tk.Button(self.root, text="Сбросить фильтр", command=self.reset_filter).grid(
            row=9, column=0, columnspan=2, pady=5
        )

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
                raise ValueError("Поле 'Описание погоды' не может быть пустым")

            # Осадки
            precipitation = self.precip_var.get()

            # Добавляем в список
            record = {
                "date": date_str,
                "temperature": temperature,
                "description": description,
                "precipitation": precipitation
            }
            self.weather_records.append(record)

            # Обновляем таблицу
            self.tree.insert("", "end", values=(
                date_str, f"{temperature}°C", description, precipitation
            ))

            # Очищаем поля
            self.date_entry.delete(0, tk.END)
            self.temp_entry.delete(0, tk.END)
            self.desc_entry.delete(0, tk.END)
            self.precip_var.set("Нет")

            # Сохраняем данные
            self.save_data()
            messagebox.showinfo("Успех", "Запись о погоде успешно добавлена!")

        except ValueError as e:
            messagebox.showerror("Ошибка ввода", str(e))
        except Exception as e:
            messagebox.showerror("Ошибка", f"Произошла непредвиденная ошибка: {str(e)}")

    def save_data(self):
        with open("weather_records.json", "w", encoding="utf-8") as f:
            json.dump(self.weather_records, f, ensure_ascii=False, indent=4)

    def load_data(self):
        if os.path.exists("weather_records.json"):
            try:
                with open("weather_records.json", "r", encoding="utf-8") as f:
                    self.weather_records = json.load(f)
                    # Заполняем таблицу при загрузке
                    for record in self.weather_records:
                        self.tree.insert(
                            "", "end",
                values=(
                    record["date"],
            f"{record['temperature']}°C",
            record["description"],
            record["precipitation"]
        )
    )
            except (json.JSONDecodeError, IOError):
                messagebox.showwarning("Предупреждение", "Не удалось загрузить данные из файла weather_records.json")
                self.weather_records = []
        else:
            self.weather_records = []

    def apply_filter(self):
        filtered = self.weather_records

        # Фильтр по дате
        filter_date_str = self.filter_date_entry.get().strip()
        if filter_date_str:
            filtered = [r for r in filtered if r["date"] == filter_date_str]

        # Фильтр по температуре
        filter_temp_str = self.filter_temp_entry.get().strip()
        if filter_temp_str:
            try:
                filter_temp = float(filter_temp_str)
                filtered = [r for r in filtered if r["temperature"] >= filter_temp]
            except ValueError:
                messagebox.showerror("Ошибка", "Неверный формат температуры для фильтра")
                return

        # Обновляем таблицу
        for item in self.tree.get_children():
            self.tree.delete(item)
        for record in filtered:
            self.tree.insert(
                "", "end",
                values=(
                    record["date"],
                    f"{record['temperature']}°C",
                    record["description"],
                    record["
