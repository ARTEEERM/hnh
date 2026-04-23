# hnh
import tkinter as tk
from tkinter import ttk, messagebox
import random
import string
import json
import os
import pyperclip

class PasswordGenerator:
    def __init__(self, root):
        self.root = root
        self.root.title("Random Password Generator")
        self.root.geometry("700x500")

        # Файл для хранения истории
        self.history_file = "history.json"
        self.history = []
        self.load_history()

        self.setup_ui()

    def setup_ui(self):
        # Панель настроек
        settings_frame = ttk.LabelFrame(self.root, text="Настройки пароля")
        settings_frame.pack(pady=10, padx=10, fill="x")

        # Ползунок длины пароля
        ttk.Label(settings_frame, text="Длина пароля (8–32):").pack(anchor="w", padx=5, pady=2)
        self.length_scale = ttk.Scale(
            settings_frame,
            from_=8,
            to=32,
            orient="horizontal"
        )
        self.length_scale.set(12)
        self.length_scale.pack(fill="x", padx=5, pady=5)

        # Чекбоксы символов
        chars_frame = ttk.Frame(settings_frame)
        chars_frame.pack(padx=5, pady=5, fill="x")

        self.digits_var = tk.BooleanVar(value=True)
        self.letters_var = tk.BooleanVar(value=True)
        self.special_var = tk.BooleanVar(value=False)

        ttk.Checkbutton(chars_frame, text="Цифры (0-9)", variable=self.digits_var).pack(anchor="w")
        ttk.Checkbutton(chars_frame, text="Буквы (a-z, A-Z)", variable=self.letters_var).pack(anchor="w")
        ttk.Checkbutton(chars_frame, text="Спецсимволы (!@#$%)", variable=self.special_var).pack(anchor="w")

        # Кнопка генерации
        generate_btn = ttk.Button(settings_frame, text="Сгенерировать пароль", command=self.generate_password)
        generate_btn.pack(pady=10)

        # Поле отображения пароля
        password_frame = ttk.LabelFrame(self.root, text="Сгенерированный пароль")
        password_frame.pack(pady=5, padx=10, fill="x")

        self.password_var = tk.StringVar()
        password_entry = ttk.Entry(password_frame, textvariable=self.password_var, font=("Courier", 12), justify="center")
        password_entry.pack(padx=5, pady=5, fill="x")

        copy_btn = ttk.Button(password_frame, text="Копировать в буфер обмена", command=self.copy_to_clipboard)
        copy_btn.pack(pady=5)

        # Таблица истории
        history_frame = ttk.LabelFrame(self.root, text="История паролей")
        history_frame.pack(pady=10, padx=10, fill="both", expand=True)

        columns = ("password", "length", "characters", "timestamp")
        self.history_tree = ttk.Treeview(history_frame, columns=columns, show="headings", height=8)

        self.history_tree.heading("password", text="Пароль")
        self.history_tree.heading("length", text="Длина")
        self.history_tree.heading("characters", text="Символы")
        self.history_tree.heading("timestamp", text="Время создания")

        self.history_tree.column("password", width=200)
        self.history_tree.column("length", width=60)
        self.history_tree.column("characters", width=150)
        self.history_tree.column("timestamp", width=180)

        scrollbar = ttk.Scrollbar(history_frame, orient="vertical", command=self.history_tree.yview)
        self.history_tree.configure(yscrollcommand=scrollbar.set)

        self.history_tree.pack(side="left", fill="both", expand=True)
        scrollbar.pack(side="right", fill="y")


        self.refresh_history()

    def validate_input(self):
        """Проверка корректности ввода"""
        length = int(self.length_scale.get())
        if length < 8 or length > 32:
            messagebox.showerror("Ошибка", "Длина пароля должна быть от 8 до 32 символов!")
            return False
        return True

    def generate_password(self):
        """Генерация случайного пароля"""
        if not self.validate_input():
            return

        length = int(self.length_scale.get())
        characters = ""

        if self.digits_var.get():
            characters += string.digits
        if self.letters_var.get():
            characters += string.ascii_letters
        if self.special_var.get():
            characters += "!@#$%^&*"

        if not characters:
            messagebox.showwarning("Предупреждение", "Выберите хотя бы один тип символов!")
            return

        password = ''.join(random.choice(characters) for _ in range(length))
        self.password_var.set(password)

        # Добавляем в историю
        char_types = []
        if self.digits_var.get(): char_types.append("цифры")
        if self.letters_var.get(): char_types.append("буквы")
        if self.special_var.get(): char_types.append("спецсимволы")

        history_entry = {
            "password": password,
            "length": length,
            "characters": ", ".join(char_types),
            "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        }
        self.history.append(history_entry)
        self.save_history()
        self.refresh_history()

    def copy_to_clipboard(self):
        """Копирование пароля в буфер обмена"""
        password = self.password_var.get()
        if password:
            pyperclip.copy(password)
            messagebox.showinfo("Успех", "Пароль скопирован в буфер обмена!")
        else:
            messagebox.showwarning("Предупреждение", "Сначала сгенерируйте пароль!")

    def save_history(self):
        """Сохранение истории в JSON"""
        try:
            with open(self.history_file, 'w', encoding='utf-8') as f:
                json.dump(self.history, f, ensure_ascii=False, indent=2)
        except Exception as e:
            messagebox.showerror("Ошибка", f"Не удалось сохранить историю: {e}")

    def load_history(self):
        """Загрузка истории из JSON"""
        try:
            if os.path.exists(self.history_file):
                with open(self.history_file, 'r', encoding='utf-8') as f:
                    self.history = json.load(f)
        except Exception as e:
            messagebox.showerror("Ошибка", f"Не удалось загрузить историю: {e}")
            self.history = []

    def refresh_history(self):
        """Обновление отображения истории"""
        for item in self.history_tree.get_children():
            self.history_tree.delete(item)

        for entry in reversed(self.history[-20:]):  # Показываем последние 20
