# expense-tracker
import tkinter as tk
from tkinter import ttk, messagebox
import requests
import json
import os

class GitHubUserFinder:
    def __init__(self, root):
        self.root = root
        self.root.title("GitHub User Finder")
        self.favorites = []
        self.load_favorites()

        # Поле поиска
        tk.Label(root, text="Имя пользователя GitHub:").grid(row=0, column=0, sticky="w", padx=10, pady=5)
        self.search_entry = tk.Entry(root, width=40)
        self.search_entry.grid(row=0, column=1, padx=10, pady=5)

        # Кнопка поиска
        tk.Button(root, text="Найти пользователя", command=self.search_user).grid(row=0, column=2, padx=5, pady=5)

        # Результаты поиска
        tk.Label(root, text="Результаты поиска:").grid(row=1, column=0, sticky="w", padx=10, pady=5)
        self.results_tree = ttk.Treeview(root, columns=("Username", "Name", "Public Repos"), show="headings", height=10)
        self.results_tree.heading("Username", text="Пользователь")
        self.results_tree.heading("Name", text="Имя")
        self.results_tree.heading("Public Repos", text="Публичных репозиториев")
        self.results_tree.column("Username", width=150)
        self.results_tree.column("Name", width=200)
        self.results_tree.column("Public Repos", width=120)
        self.results_tree.grid(row=2, column=0, columnspan=3, padx=10, pady=5, sticky="nsew")

        # Кнопки для избранного
        tk.Button(root, text="Добавить в избранное", command=self.add_to_favorites).grid(row=3, column=0, padx=10, pady=10)

        # Список избранного
        tk.Label(root, text="Избранное:").grid(row=4, column=0, sticky="w", padx=10, pady=5)
        self.favorites_tree = ttk.Treeview(root, columns=("Username", "Name"), show="headings", height=5)
        self.favorites_tree.heading("Username", text="Пользователь")
        self.favorites_tree.heading("Name", text="Имя")
        self.favorites_tree.column("Username", width=250)
        self.favorites_tree.column("Name", width=300)
        self.favorites_tree.grid(row=5, column=0, columnspan=3, padx=10, pady=5, sticky="nsew")

        # Кнопки сохранения/загрузки
        tk.Button(root, text="Сохранить избранное", command=self.save_favorites).grid(row=6, column=0, padx=10, pady=5)
        tk.Button(root, text="Загрузить избранное", command=self.load_favorites).grid(row=6, column=1, pady=5)

        self.update_favorites_table()

    def validate_input(self):
        username = self.search_entry.get().strip()
        if not username:
            messagebox.showerror("Ошибка", "Поле поиска не должно быть пустым!")
            return False
        return True

    def search_user(self):
        if not self.validate_input():
            return

        username = self.search_entry.get().strip()

        try:
            response = requests.get(f"https://api.github.com/users/{username}")
            if response.status_code == 200:
                user_data = response.json()
                self.display_search_result(user_data)
            else:
                messagebox.showerror("Ошибка", f"Пользователь '{username}' не найден!")
        except requests.exceptions.RequestException as e:
            messagebox.showerror("Ошибка сети", f"Не удалось подключиться к GitHub API: {e}")

    def display_search_result(self, user_data):
        for item in self.results_tree.get_children():
            self.results_tree.delete(item)

        name = user_data.get("name", "Не указано")
        username = user_data["login"]
        public_repos = user_data["public_repos"]

        self.results_tree.insert("", "end", values=(username, name, public_repos))

    def add_to_favorites(self):
        selection = self.results_tree.selection()
        if not selection:
            messagebox.showwarning("Предупреждение", "Выберите пользователя из результатов поиска!")
            return

        item = selection[0]
        values = self.results_tree.item(item, "values")
        favorite = {
            "username": values[0],
            "name": values[1]
        }

        if favorite not in self.favorites:
            self.favorites.append(favorite)
            self.update_favorites_table()
            messagebox.showinfo("Успех", f"{values[0]} добавлен в избранное!")
        else:
            messagebox.showinfo("Информация", f"{values[0]} уже в избранном!")

    def update_favorites_table(self):
        for item in self.favorites_tree.get_children():
            self.favorites_tree.delete(item)
        for fav in self.favorites:
            self.favorites_tree.insert("", "end", values=(fav["username"], fav["name"]))

    def save_favorites(self):
        with open("favorites.json", "w", encoding="utf-8") as f:
            json.dump(self.favorites, f, ensure_ascii=False, indent=4)
        messagebox.showinfo("Успех", "Избранное сохранено в favorites.json")

    def load_favorites(self):
        if os.path.exists("favorites.json"):
            with open("favorites.json", "r", encoding="utf-8") as f:
                self.favorites = json.load(f)
            self.update_favorites_table()
            messagebox.showinfo("Успех", "Избранное загружено из favorites.json")
        else:
            messagebox.showwarning("Предупреждение", "Файл favorites.json не найден!")

    def load_favorites(self):
        if os.path.exists("favorites.json"):
            with open("favorites.json", "r", encoding="utf-8") as f:
                self.favorites = json.load(f)

if __name__ == "__main__":
    root = tk.Tk()
    app = GitHubUserFinder(root)
    root.mainloop()
