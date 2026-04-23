import tkinter as tk
from tkinter import ttk, messagebox, scrolledtext
import json
import random
import os

class QuoteGenerator:
    def __init__(self, root):
        self.root = root
        self.root.title("Random Quote Generator - Генератор случайных цитат")
        self.quotes = self.load_predefined_quotes()
        self.history = []
        self.create_widgets()
        self.load_history()

    def load_predefined_quotes(self):
        """Загружаем предопределённые цитаты"""
        return [
            {"text": "Жизнь — это то, что происходит с тобой, пока ты строишь другие планы.",
             "author": "Джон Леннон", "topic": "Жизнь"},
            {"text": "Единственный способ сделать великую работу — любить то, что ты делаешь.",
             "author": "Стив Джобс", "topic": "Работа"},
            {"text": "Будущее принадлежит тем, кто верит в красоту своей мечты.",
             "author": "Элеонора Рузвельт", "topic": "Мотивация"},
            {"text": "В середине трудности кроется возможность.",
             "author": "Альберт Эйнштейн", "topic": "Трудности"},
            {"text": "Успех — это способность идти от неудачи к неудаче без потери энтузиазма.",
             "author": "Уинстон Черчилль", "topic": "Успех"}
        ]

    def create_widgets(self):
        # Кнопка генерации
        tk.Button(self.root, text="Сгенерировать цитату",
                   command=self.generate_quote, font=("Arial", 12)).pack(pady=10)

        # Отображение цитаты
        self.quote_display = scrolledtext.ScrolledText(
            self.root, width=60, height=5, font=("Arial", 10))
        self.quote_display.pack(pady=5, padx=10, fill="x")

        # История цитат
        tk.Label(self.root, text="История сгенерированных цитат:",
                  font=("Arial", 12, "bold")).pack(anchor="w", padx=10, pady=(10, 5))

        self.history_list = tk.Listbox(self.root, height=10, width=70)
        self.history_list.pack(pady=5, padx=10, fill="both", expand=True)

        # Фильтрация
        filter_frame = tk.Frame(self.root)
        filter_frame.pack(pady=10, padx=10, fill="x")

        tk.Label(filter_frame, text="Фильтр по автору:").grid(row=0, column=0, padx=5, sticky="w")
        self.author_filter = ttk.Combobox(filter_frame, values=["Все"] +
                                           list(set(q["author"] for q in self.quotes)))
        self.author_filter.set("Все")
        self.author_filter.grid(row=0, column=1, padx=5)

        tk.Label(filter_frame, text="Фильтр по теме:").grid(row=1, column=0, padx=5, pady=5, sticky="w")
        self.topic_filter = ttk.Combobox(filter_frame, values=["Все"] +
                         list(set(q["topic"] for q in self.quotes)))
        self.topic_filter.set("Все")
        self.topic_filter.grid(row=1, column=1, padx=5, pady=5)

        tk.Button(filter_frame, text="Применить фильтр",
                  command=self.apply_filter).grid(row=2, column=0, pady=5)
        tk.Button(filter_frame, text="Сбросить фильтр",
                  command=self.reset_filter).grid(row=2, column=1, pady=5)

        # Добавление новых цитат
        add_frame = tk.Frame(self.root)
        add_frame.pack(pady=10, padx=10, fill="x")

        tk.Label(add_frame, text="Добавить новую цитату:").grid(row=0, column=0,
                  columnspan=2, padx=5, sticky="w")

        tk.Label(add_frame, text="Текст:").grid(row=1, column=0, padx=5, sticky="w")
        self.new_text = scrolledtext.ScrolledText(add_frame, width=40, height=3)
        self.new_text.grid(row=1, column=1, padx=5, pady=2)

        tk.Label(add_frame, text="Автор:").grid(row=2, column=0, padx=5, pady=2, sticky="w")
        self.new_author = tk.Entry(add_frame, width=40)
        self.newauthor.grid(row=2, column=1, padx=5, pady=2)

        tk.Label(add_frame, text="Тема:").grid(row=3, column=0, padx=5, pady=2, sticky="w")
        self.newtopic = tk.Entry(add_frame, width=40)
        self.newtopic.grid(row=3, column=1, padx=5, pady=2)

        tk.Button(add_frame, text="Добавить цитату",
                  command=self.add_new_quote).grid(row=4, column=0, columnspan=2, pady=5)

    def generate_quote(self):
        if not self.quotes:
            messagebox.showwarning("Предупреждение", "Нет доступных цитат!")
            return

        quote = random.choice(self.quotes)
        display_text = f'"{quote["text"]}"\n— {quote["author"]} ({quote["topic"]})'

        self.quote_display.delete(1.0, tk.END)
        self.quote_display.insert(1.0, display_text)

        # Добавляем в историю
        self.history.append(quote)
        self.update_history_display()
        self.save_history()

    def update_history_display(self):
        self.history_list.delete(0, tk.END)
        for i, quote in enumerate(reversed(self.history[-20:]), 1):  # Последние 20 цитат
            display = f"{i}. {quote['author']}: {quote['text'][:50]}..."
            self.history_list.insert(tk.END, display)
    def apply_filter(self):
        filtered = self.history

        author_filter = self.author_filter.get()
        topic_filter = self.topic_filter.get()

        if author_filter != "Все":
            filtered = [q for q in filtered if q["author"] == author_filter]
        if topic_filter != "Все":
            filtered = [q for q in filtered if q["topic"] == topic_filter]


        self.history_list.delete(0, tk.END)
        for i, quote in enumerate(filtered, 1):
            display = f"{i}. {quote['author']}: {quote['text'][:50]}..."
            self.history_list.insert(tk.END, display)
    def reset_filter(self):
        self.author_filter.set("Все")
        self.topic_filter.set("Все")
        self.update_history_display()
    def add_new_quote(self):
        text = self.new_text.get(1.0, tk.END).strip()
        author = self.newauthor.get().strip()
        topic = self.newtopic.get().strip()

        if not text or not author or not topic:
            messagebox.showerror("Ошибка", "Все поля должны быть заполнены!")
            return

        new_quote = {"text": text, "author": author, "topic": topic}
        self.quotes.append(new_quote)

        # Обновляем фильтры
        authors = list(set(q["author"] for q in self.quotes))
        topics = list(set(q["topic"] for q in self.quotes))

        self.author_filter["values"] = ["Все"] + authors
        self.topic_filter["values"] = ["
