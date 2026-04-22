# expense-tracker
import tkinter as tk
from tkinter import ttk, messagebox, scrolledtext
import json
import random
import os

class QuoteGenerator:
    def __init__(self, root):
        self.root = root
        self.root.title("Random Quote Generator - Генератор случайных цитат")
        self.quotes = self.load_quotes()
        self.history = self.load_history()
        self.create_widgets()

    def create_widgets(self):
        # Кнопка генерации
        tk.Button(self.root, text="Сгенерировать цитату", command=self.generate_quote).pack(pady=10)

        # Отображение цитаты
        self.quote_text = scrolledtext.ScrolledText(
            self.root, width=60, height=5, wrap=tk.WORD
        )
        self.quote_text.pack(padx=10, pady=5)
        self.quote_text.config(state=tk.DISABLED)

        # История цитат
        tk.Label(self.root, text="История цитат:").pack(anchor="w", padx=10)
        self.history_listbox = tk.Listbox(self.root, width=60, height=10)
        self.history_listbox.pack(padx=10, pady=5, fill=tk.BOTH, expand=True)

        # Фильтрация
        filter_frame = tk.Frame(self.root)
        filter_frame.pack(pady=5)

        tk.Label(filter_frame, text="Фильтр по автору:").grid(row=0, column=0, padx=5)
        self.author_filter = ttk.Combobox(filter_frame, values=["Все"] + self.get_authors())
        self.author_filter.set("Все")
        self.author_filter.grid(row=0, column=1, padx=5)

        tk.Label(filter_frame, text="Фильтр по теме:").grid(row=1, column=0, padx=5)
        self.topic_filter = ttk.Combobox(filter_frame, values=["Все"] + self.getTopics())
        self.topic_filter.set("Все")
        self.topic_filter.grid(row=1, column=1, padx=5)

        tk.Button(filter_frame, text="Применить фильтр", command=self.apply_filter).grid(row=2, column=0, pady=5)
        tk.Button(filter_frame, text="Сбросить фильтр", command=self.reset_filter).grid(row=2, column=1, pady=5)

        # Добавление новых цитат
        add_frame = tk.Frame(self.root)
        add_frame.pack(pady=10)


        tk.Label(add_frame, text="Текст цитаты:").grid(row=0, column=0, padx=5, sticky="w")
        self.new_quote_entry = tk.Entry(add_frame, width=40)
        self.new_quote_entry.grid(row=0, column=1, padx=5)

        tk.Label(add_frame, text="Автор:").grid(row=1, column=0, padx=5, sticky="w")
        self.new_author_entry = tk.Entry(add_frame, width=40)
        self.new_author_entry.grid(row=1, column=1, padx=5)

        tk.Label(add_frame, text="Тема:").grid(row=2, column=0, padx=5, sticky="w")
        self.newtopic_entry = tk.Entry(add_frame, width=40)
        self.newtopic_entry.grid(row=2, column=1, padx=5)

        tk.Button(add_frame, text="Добавить цитату", command=self.add_quote).grid(row=3, column=0, columnspan=2, pady=5)

        self.update_history_display()

    def load_quotes(self):
        default_quotes = [
            {"text": "Знание — сила.", "author": "Фрэнсис Бэкон", "topic": "Знание"},
            {"text": "Быть или не быть — вот в чём вопрос.", "author": "Уильям Шекспир", "topic": "Философия"},
            {"text": "Через тернии к звёздам.", "author": "Сенека", "topic": "Мотивация"},
            {"text": "Я мыслю, следовательно, существую.", "author": "Рене Декарт", "topic": "Философия"}
        ]
        if os.path.exists("quotes.json"):
            try:
                with open("quotes.json", "r", encoding="utf-8") as f:
                    return json.load(f)
            except (json.JSONDecodeError, IOError):
                return default_quotes
        return default_quotes

    def save_quotes(self):
        with open("quotes.json", "w", encoding="utf-8") as f:
            json.dump(self.quotes, f, ensure_ascii=False, indent=4)

    def load_history(self):
        if os.path.exists("history.json"):
            try:
                with open("history.json", "r", encoding="utf-8") as f:
                    return json.load(f)
            except (json.JSONDecodeError, IOError):
                return []
        return []

    def save_history(self):
        with open("history.json", "w", encoding="utf-8") as f:
            json.dump(self.history, f, ensure_ascii=False, indent=4)

    def generate_quote(self):
        if not self.quotes:
            messagebox.showwarning("Предупреждение", "Список цитат пуст!")
            return
        quote = random.choice(self.quotes)
        display_text = f'"{quote["text"]}"\n— {quote["author"]} ({quote["topic"]})'
        self.quote_text.config(state=tk.NORMAL)
        self.quote_text.delete(1.0, tk.END)
        self.quote_text.insert(1.0, display_text)
        self.quote_text.config(state=tk.DISABLED)
        # Добавляем в историю
        self.history.append(quote)
        self.save_history()
        self.update_history_display()

    def add_quote(self):
        text = self.new_quote_entry.get().strip()
        author = self.newauthor_entry.get().strip()
        topic = self.newtopic_entry.get().strip()

        if not text:
            messagebox.showerror("Ошибка", "Текст цитаты не может быть пустым")
            return
        if not author:
            messagebox.showerror("Ошибка", "Автор не может быть пустым")
            return
        if not topic:
            messagebox.showerror("Ошибка", "Тема не может быть пустой")
            return

        new_quote = {"text": text, "author": author, "topic": topic}
        self.quotes.append(new_quote)
        self.save_quotes()
        self.new_quote_entry.delete(0, tk.END)
        self.newauthor_entry.delete(0, tk.END)
        self.newtopic_entry.delete(0, tk.END)
        messagebox.showinfo("Успех", "Цитата успешно добавлена!")

    def update_history_display(self):
        self.history_listbox.delete(0, tk.END)
        for quote in self.history:
            self.history_listbox.insert(tk.END, f'{quote["author"]}: "{quote["text"][:50]}..."')

    def apply_filter(self):
        filtered = self.history
        author_filter = self.author_filter.get()
        topic_filter = self.topic_filter.get()


        if author_filter != "Все":
            filtered = [q for q in filtered if q["author"] == author_filter]
        if topic_filter != "Все":
            filtered = [q for q in filtered if q["topic"] == topic_filter]

        self.history_listbox.delete(0, tk.END)
        for quote in filtered:
            self.history_listbox.insert(tk.END, f'{quote["author"]}: "{quote
