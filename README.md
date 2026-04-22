# expense-tracker
import tkinter as tk
from tkinter import ttk, messagebox, scrolledtext
import requests
import json
import os

class GitHubUserFinder:
    def __init__(self, root):
        self.root = root
        self.root.title("GitHub User Finder")
        self.favorites = []
        self.current_user_data = None  # Храним данные текущего пользователя для добавления в избранное
        self.create_widgets()
        self.load_favorites()  # Загружаем избранное после создания виджетов

    def create_widgets(self):
        # Поле поиска
        search_frame = tk.Frame(self.root)
        search_frame.pack(pady=10, padx=10, fill="x")

        tk.Label(search_frame, text="Имя пользователя GitHub:").pack(side="left")
        self.search_entry = tk.Entry(search_frame, width=30)
        self.search_entry.pack(side="left", padx=5)
        tk.Button(search_frame, text="Найти", command=self.search_user).pack(side="left")

        # Результаты поиска
        result_frame = tk.Frame(self.root)
        result_frame.pack(pady=10, padx=10, fill="both", expand=True)

        tk.Label(result_frame, text="Результаты поиска:").pack(anchor="w")
        self.results_text = scrolledtext.ScrolledText(
            result_frame, width=60, height=15, wrap=tk.WORD
        )
        self.results_text.pack(fill="both", expand=True, pady=5)

        # Кнопки действий
        button_frame = tk.Frame(self.root)
        button_frame.pack(pady=5)
        tk.Button(button_frame, text="Добавить в избранное", command=self.add_to_favorites).pack(side="left", padx=5)
        tk.Button(button_frame, text="Показать избранное", command=self.show_favorites).pack(side="left", padx=5)

        # Список избранного
        favorites_frame = tk.Frame(self.root)
        favorites_frame.pack(pady=10, padx=10, fill="both", expand=True)
        tk.Label(favorites_frame, text="Избранное:").pack(anchor="w")
        self.favorites_listbox = tk.Listbox(favorites_frame, width=60, height=8)
        self.favorites_listbox.pack(fill="both", expand=True, pady=5)

    def search_user(self):
        username = self.search_entry.get().strip()
        if not username:
            messagebox.showerror("Ошибка", "Поле поиска не может быть пустым")
            return

        try:
            response = requests.get(f"https://api.github.com/users/{username}")
            if response.status_code == 200:
                self.current_user_data = response.json()
                self.display_user_info(self.current_user_data)
            elif response.status_code == 404:
                messagebox.showwarning("Предупреждение", "Пользователь не найден")
                self.results_text.delete(1.0, tk.END)
                self.results_text.insert(1.0, "Пользователь не найден.")
            else:
                messagebox.showerror("Ошибка", f"Ошибка API: {response.status_code}")
                self.results_text.delete(1.0, tk.END)
        except requests.exceptions.RequestException as e:
            messagebox.showerror("Ошибка сети", f"Не удалось подключиться к GitHub API: {str(e)}")
            self.results_text.delete(1.0, tk.END)

    def display_user_info(self, user_data):
        info = f"""
Имя: {user_data.get('name', 'Не указано')}
Логин: {user_data['login']}
Компания: {user_data.get('company', 'Не указана')}
Местоположение: {user_data.get('location', 'Не указано')}
Публичные репозитории: {user_data.get('public_repos', 0)}
Подписчики: {user_data.get('followers', 0)}
Следит: {user_data.get('following', 0)}
Биография: {user_data.get('bio', 'Не указана')}
Профиль: {user_data['html_url']}
        """
        self.results_text.delete(1.0, tk.END)
        self.results_text.insert(1.0, info)

    def add_to_favorites(self):
        if not self.current_user_data:
            messagebox.showwarning("Предупреждение", "Сначала найдите пользователя")
            return

        username = self.current_user_data['login']
        if username not in self.favorites:
            self.favorites.append(username)
            self.save_favorites()
            self.update_favorites_display()
            messagebox.showinfo("Успех", f"Пользователь {username} добавлен в избранное!")
        else:
            messagebox.showinfo("Информация", "Этот пользователь уже в избранном")

    def show_favorites(self):
        if not self.favorites:
            messagebox.showinfo("Избранное", "Список избранного пуст")
            return

        favorites_info = []
        for username in self.favorites:
            try:
                response = requests.get(f"https://api.github.com/users/{username}")
                if response.status_code == 200:
                    user_data = response.json()
                    favorites_info.append(f"{user_data.get('name', username)} ({user_data['login']})")
                else:
                    favorites_info.append(username)
            except:
                favorites_info.append(username)

        self.results_text.delete(1.0, tk.END)
        self.results_text.insert(1.0, "\n".join(favorites_info))

    def load_favorites(self):
        if os.path.exists("favorites.json"):
            try:
                with open("favorites.json", "r", encoding="utf-8") as f:
                    self.favorites = json.load(f)
                self.update_favorites_display()
            except (json.JSONDecodeError, IOError) as e:
                print(f"Предупреждение: не удалось загрузить избранное: {e}")
                self.favorites = []
        else:
            self.favorites = []

    def save_favorites(self):
        with open("favorites.json", "w", encoding="utf-8") as f:
            json.dump(self.favorites, f, ensure_ascii=False, indent=4)


    def update_favorites_display(self):
        self.favorites_listbox.delete(0, tk.END)
        for user in self.favorites:
            self.favorites_listbox.insert(tk.END, user)

def main():
    root = tk.Tk()
    app = GitHubUserFinder(root)
    root.mainloop()

if __name__ == "__main__":
    main()
