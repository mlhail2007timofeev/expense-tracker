# expense-tracker
import tkinter as tk
from tkinter import ttk, messagebox
import json
import os
import random

class QuoteGenerator:
    def __init__(self, root):
        self.root = root
        self.root.title("Random Quote Generator")
        self.history = []
        self.quotes = [
            {"text": "Программирование — это не наука, это ремесло.", "author": "Неизвестный", "topic": "Программирование"},
            {"text": "Код должен быть написан для людей, а не для компьютеров.", "author": "Мартин Фаулер", "topic": "Разработка"},
            {"text": "Простота — это высшая степень совершенства.", "author": "Леонардо да Винчи", "topic": "Философия"},
            {"text": "Лучший код — это тот, который не нужно писать.", "author": "Неизвестный", "topic": "Программирование"},
            {"text": "Успех — это способность идти от неудачи к неудаче, не теряя энтузиазма.", "author": "Уинстон Черчилль", "topic": "Мотивация"}
        ]
        self.load_data()

        # Поле для отображения цитаты
        self.quote_label = tk.Label(root, text="Нажмите 'Сгенерировать цитату'", wraplength=400, justify="center", font=("Arial", 12))
        self.quote_label.grid(row=0, column=0, columnspan=2, padx=20, pady=10)

        self.author_label = tk.Label(root, text="", font=("Arial", 10, "italic"))
        self.author_label.grid(row=1, column=0, columnspan=2, padx=20, pady=5)

        self.topic_label = tk.Label(root, text="", font=("Arial", 9))
        self.topic_label.grid(row=2, column=0, columnspan=2, padx=20, pady=5)


        # Кнопка генерации
        tk.Button(root, text="Сгенерировать цитату", command=self.generate_quote).grid(row=3, column=0, columnspan=2, pady=10)


        # Таблица истории
        tk.Label(root, text="История цитат:").grid(row=4, column=0, sticky="w", padx=10, pady=5)
        self.history_tree = ttk.Treeview(root, columns=("Quote", "Author", "Topic"), show="headings", height=8)
        self.history_tree.heading("Quote", text="Цитата")
        self.history_tree.heading("Author", text="Автор")
        self.history_tree.heading("Topic", text="Тема")
        self.history_tree.column("Quote", width=250)
        self.history_tree.column("Author", width=120)
        self.history_tree.column("Topic", width=100)
        self.history_tree.grid(row=5, column=0, columnspan=2, padx=10, pady=5, sticky="nsew")


        # Фильтры
        tk.Label(root, text="Фильтр по автору:").grid(row=6, column=0, sticky="w", padx=10, pady=5)
        self.author_filter = ttk.Combobox(root)
        self.author_filter.grid(row=6, column=1, padx=10, pady=5, sticky="w")


        tk.Label(root, text="Фильтр по теме:").grid(row=7, column=0, sticky="w", padx=10, pady=5)
        self.topic_filter = ttk.Combobox(root)
        self.topic_filter.grid(row=7, column=1, padx=10, pady=5, sticky="w")

        tk.Button(root, text="Применить фильтр", command=self.apply_filter).grid(row=8, column=0, pady=5)
        tk.Button(root, text="Сбросить фильтр", command=self.reset_filter).grid(row=8, column=1, pady=5)

        # Кнопки сохранения/загрузки
        tk.Button(root, text="Сохранить историю", command=self.save_history).grid(row=9, column=0, pady=5)
        tk.Button(root, text="Загрузить историю", command=self.load_history).grid(row=9, column=1, pady=5)

        self.update_filters()
        self.update_history_table()

    def generate_quote(self):
        if not self.quotes:
            messagebox.showwarning("Предупреждение", "Список цитат пуст!")
            return
        quote = random.choice(self.quotes)
        self.history.append(quote)
        self.display_quote(quote)
        self.update_history_table()

    def display_quote(self, quote):
        self.quote_label.config(text=quote["text"])
        self.author_label.config(text=f"— {quote['author']}")
        self.topic_label.config(text=f"Тема: {quote['topic']}")

    def update_history_table(self):
        for item in self.history_tree.get_children():
            self.history_tree.delete(item)
        for quote in self.history:
            self.history_tree.insert("", "end", values=(quote["text"], quote["author"], quote["topic"]))


    def update_filters(self):
        authors = list(set([q["author"] for q in self.quotes]))
        topics = list(set([q["topic"] for q in self.quotes]))
        self.author_filter["values"] = ["Все"] + authors
        self.topic_filter["values"] = ["Все"] + topics
        self.author_filter.set("Все")
        self.topic_filter.set("Все")

    def apply_filter(self):
        author_filter = self.author_filter.get()
        topic_filter = self.topic_filter.get()

        filtered_history = self.history

        if author_filter != "Все":
            filtered_history = [q for q in filtered_history if q["author"] == author_filter]

        if topic_filter != "Все":
            filtered_history = [q for q in filtered_history if q["topic"] == topic_filter]


        for item in self.history_tree.get_children():
            self.history_tree.delete(item)

        for quote in filtered_history:
            self.history_tree.insert("", "end", values=(quote["text"], quote["author"], quote["topic"]))

    def reset_filter(self):
        self.author_filter.set("Все")
        self.topic_filter.set("Все")
        self.update_history_table()

    def save_history(self):
        with open("quote_history.json", "w", encoding="utf-8") as f:
            json.dump(self.history, f, ensure_ascii=False, indent=4)
        messagebox.showinfo("Успех", "История сохранена в quote_history.json")

    def load_history(self):
        if os.path.exists("quote_history.json"):
            with open("quote_history.json", "r", encoding="utf-8") as f:
                self.history = json.load(f)
            self.update_history_table()
            messagebox.showinfo("Успех", "История загружена из quote_history.json")
        else:
            messagebox.showwarning("Предупреждение", "Файл quote_history.json не найден!")


    def load_data(self):
        if os.path.exists("quotes.json"):
            with open("quotes.json", "r", encoding="utf-8") as f:
                self.quotes = json.load(f)

if __name__ == "__main__":
    root = tk.Tk()
    app = QuoteGenerator(root)
    root.mainloop()
