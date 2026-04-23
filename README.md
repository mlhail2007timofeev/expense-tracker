# expense-tracker
import tkinter as tk
from tkinter import ttk, messagebox
import json
from datetime import datetime
import os

class TrainingPlanner:
    def __init__(self, root):
        self.root = root
        self.root.title("Training Planner - План тренировок")
        self.trainings = []
        self.create_widgets()
        self.load_data()

    def create_widgets(self):
        # Форма ввода
        input_frame = tk.Frame(self.root)
        input_frame.pack(pady=10, padx=10, fill="x")

        # Дата
        tk.Label(input_frame, text="Дата (ГГГГ-ММ-ДД):").grid(row=0, column=0, padx=5, pady=5, sticky="w")
        self.date_entry = tk.Entry(input_frame, width=20)
        self.date_entry.grid(row=0, column=1, padx=5, pady=5)

        # Тип тренировки
        tk.Label(input_frame, text="Тип тренировки:").grid(row=1, column=0, padx=5, pady=5, sticky="w")
        self.type_entry = ttk.Combobox(input_frame, values=[
            "Бег", "Плавание", "Силовая", "Йога", "Велоспорт", "Кроссфит"
        ], width=17)
        self.type_entry.grid(row=1, column=1, padx=5, pady=5)

        # Длительность
        tk.Label(input_frame, text="Длительность (мин):").grid(row=2, column=0, padx=5, pady=5, sticky="w")
        self.duration_entry = tk.Entry(input_frame, width=20)
        self.duration_entry.grid(row=2, column=1, padx=5, pady=5)

        # Кнопка добавления
        tk.Button(input_frame, text="Добавить тренировку", command=self.add_training).grid(
            row=3, column=0, columnspan=2, pady=10
        )

        # Таблица тренировок
        table_frame = tk.Frame(self.root)
        table_frame.pack(pady=10, padx=10, fill="both", expand=True)

        columns = ("Дата", "Тип тренировки", "Длительность (мин)")
        self.tree = ttk.Treeview(table_frame, columns=columns, show="headings", height=10)
        for col in columns:
            self.tree.heading(col, text=col)
            self.tree.column(col, width=150)
        self.tree.pack(fill="both", expand=True)

        # Фильтрация
        filter_frame = tk.Frame(self.root)
        filter_frame.pack(pady=10, padx=10, fill="x")

        tk.Label(filter_frame, text="Фильтр по типу:").grid(row=0, column=0, padx=5, pady=5, sticky="w")
        self.filter_type = ttk.Combobox(filter_frame, values=["Все"] + [
            "Бег", "Плавание", "Силовая", "Йога", "Велоспорт", "Кроссфит"
        ])
        self.filter_type.set("Все")
        self.filter_type.grid(row=0, column=1, padx=5, pady=5)

        tk.Label(filter_frame, text="Фильтр по дате:").grid(row=1, column=0, padx=5, pady=5, sticky="w")
        self.filter_date = tk.Entry(filter_frame, width=20)
        self.filter_date.grid(row=1, column=1, padx=5, pady=5)

        tk.Button(filter_frame, text="Применить фильтр", command=self.apply_filter).grid(row=2, column=0, pady=5)
        tk.Button(filter_frame, text="Сбросить фильтр", command=self.reset_filter).grid(row=2, column=1, pady=5)

    def add_training(self):
        try:
            # Валидация даты
            date_str = self.date_entry.get().strip()
            if not date_str:
                raise ValueError("Поле 'Дата' не может быть пустым")
            try:
                date = datetime.strptime(date_str, "%Y-%m-%d")
            except ValueError:
                raise ValueError("Неверный формат даты. Используйте ГГГГ-ММ-ДД")

            # Валидация типа тренировки
            training_type = self.type_entry.get().strip()
            if not training_type:
                raise ValueError("Выберите тип тренировки")

            # Валидация длительности
            duration_str = self.duration_entry.get().strip()
            if not duration_str:
                raise ValueError("Поле 'Длительность' не может быть пустым")
            duration = float(duration_str)
            if duration <= 0:
                raise ValueError("Длительность должна быть положительным числом")

            # Добавляем запись
            training = {
                "date": date_str,
                "type": training_type,
                "duration": duration
            }
            self.trainings.append(training)

            # Обновляем таблицу
            self.tree.insert("", "end", values=(
                date_str, training_type, f"{duration} мин"
            ))

            # Очищаем поля
            self.date_entry.delete(0, tk.END)
            self.type_entry.set("")
            self.duration_entry.delete(0, tk.END)

            # Сохраняем данные
            self.save_data()
            messagebox.showinfo("Успех", "Тренировка успешно добавлена!")

        except ValueError as e:
            messagebox.showerror("Ошибка ввода", str(e))
        except Exception as e:
            messagebox.showerror("Ошибка", f"Произошла непредвиденная ошибка: {str(e)}")


    def save_data(self):
        with open("trainings.json", "w", encoding="utf-8") as f:
            json.dump(self.trainings, f, ensure_ascii=False, indent=4)

    def load_data(self):
        if os.path.exists("trainings.json"):
            try:
                with open("trainings.json", "r", encoding="utf-8") as f:
                    self.trainings = json.load(f)
                # Заполняем таблицу при загрузке
                for training in self.trainings:
                    self.tree.insert("", "end", values=(
                training["date"],
                training["type"],
                f"{training['duration']} мин"
            ))
            except (json.JSONDecodeError, IOError) as e:
                print(f"Предупреждение: не удалось загрузить данные: {e}")
        else:
            self.trainings = []

    def apply_filter(self):
        filtered = self.trainings

        # Фильтр по типу тренировки
        filter_type = self.filter_type.get()
        if filter_type != "Все":
            filtered = [t for t in filtered if t["type"] == filter_type]

        # Фильтр по дате
        filter_date = self.filter_date.get().strip()
        if filter_date:
            filtered = [t for t in filtered if t["date"] == filter_date]

        # Обновляем таблицу
        for item in self.tree.get_children():
            self.tree.delete(item)
        for training in filtered:
            self.tree.insert(
                "", "end",
                values=(training["date"], training["type"], f"{training['duration']} мин")
            )

    def reset_filter(self):
        # Сбрасываем фильтры
        self.filter_type.set("Все")
        self.filter_date.delete(0, tk.END)
        # Показываем все записи
        for item in self.tree.get_children():
            self.tree.delete(item)
        for training in self.trainings:
            self.tree.insert(
                "", "end",
                values=(training["date"], training["type"], f"{training['duration']} мин")
            )

def main():
    root = tk.Tk()
    app = Training
