# jkjk
nm,
import tkinter as tk
from tkinter import ttk, messagebox
import random
import json
import os

# Список предопределённых цитат
quotes = [
    {
        "text": "Знание — сила",
        "author": "Фрэнсис Бэкон",
        "topic": "Философия"
    },
    {
        "text": "Быть или не быть — вот в чём вопрос",
        "author": "Уильям Шекспир",
        "topic": "Литература"
    },
    {
        "text": "Программирование — это искусство решать проблемы",
        "author": "Неизвестный",
        "topic": "Программирование"
    },
    {
        "text": "Жизнь — это то, что происходит с тобой, пока ты строишь другие планы",
        "author": "Джон Леннон",
        "topic": "Жизнь"
    },
    {
        "text": "Успех — это способность идти от неудачи к неудаче, не теряя энтузиазма",
        "author": "Уинстон Черчилль",
        "topic": "Мотивация"
    }
]

class QuoteGenerator:
    def __init__(self, root):
        self.root = root
        self.root.title("Random Quote Generator")
        self.root.geometry("600x500")

        # Загрузка истории или создание пустой
        self.history = self.load_history()

        self.setup_ui()

    def setup_ui(self):
        # Верхняя часть — кнопка генерации
        top_frame = ttk.Frame(self.root)
        top_frame.pack(pady=10)

        ttk.Button(top_frame, text="Сгенерировать цитату",
                   command=self.generate_quote).pack(side="left", padx=5)

        # Поле отображения цитаты
        self.quote_text = tk.Text(self.root, height=4, wrap="word")
        self.quote_text.pack(padx=20, pady=10, fill="x")

        # Фильтры
        filter_frame = ttk.LabelFrame(self.root, text="Фильтры")
        filter_frame.pack(padx=20, pady=5, fill="x")

        ttk.Label(filter_frame, text="Автор:").grid(row=0, column=0, padx=5, pady=5)
        self.author_filter = ttk.Combobox(filter_frame)
        self.author_filter.grid(row=0, column=1, padx=5, pady=5)

        ttk.Label(filter_frame, text="Тема:").grid(row=0, column=2, padx=5, pady=5)
        self.topic_filter = ttk.Combobox(filter_frame)
        self.topic_filter.grid(row=0, column=3, padx=5, pady=5)

        ttk.Button(filter_frame, text="Применить",
                   command=self.apply_filters).grid(row=0, column=4, padx=5, pady=5)

        # История цитат
        history_frame = ttk.LabelFrame(self.root, text="История цитат")
        history_frame.pack(padx=20, pady=10, fill="both", expand=True)

        self.history_list = tk.Listbox(history_frame)
        scrollbar = ttk.Scrollbar(history_frame, orient="vertical",
                                   command=self.history_list.yview)
        self.history_list.configure(yscrollcommand=scrollbar.set)

        self.history_list.pack(side="left", fill="both", expand=True)
        scrollbar.pack(side="right", fill="y")

        # Обновление фильтров
        self.update_filters()
        self.update_history_display()

    def generate_quote(self):
        filtered_quotes = self.get_filtered_quotes()
        if not filtered_quotes:
            messagebox.showwarning("Предупреждение", "Нет цитат для отображения")
            return

        quote = random.choice(filtered_quotes)
        display_text = f"{quote['text']}\n— {quote['author']} ({quote['topic']})"

        self.quote_text.delete(1.0, tk.END)
        self.quote_text.insert(1.0, display_text)

        # Добавляем в историю
        self.history.append(quote)
        self.save_history()
        self.update_history_display()

    def get_filtered_quotes(self):
        author = self.author_filter.get()
        topic = self.topic_filter.get()

        filtered = quotes
        if author:
            filtered = [q for q in filtered if q['author'] == author]
        if topic:
            filtered = [q for q in filtered if q['topic'] == topic]
        return filtered

    def update_history_display(self):
        self.history_list.delete(0, tk.END)
        for quote in self.history[-20:]:  # Последние 20 цитат
            self.history_list.insert(tk.END, f"{quote['author']}: {quote['text'][:50]}...")

    def load_history(self):
        if os.path.exists("history.json"):
            with open("history.json", "r", encoding="utf-8") as f:
                return json.load(f)
        return []

    def save_history(self):
        with open("history.json", "w", encoding="utf-8") as f:
            json.dump(self.history, f, ensure_ascii=False, indent=2)

    def update_filters(self):
        authors = sorted(set(q['author'] for q in quotes))
        topics = sorted(set(q['topic'] for q in quotes))

        self.author_filter['values'] = [""] + authors
        self.topic_filter['values'] = [""] + topics

    def apply_filters(self):
        self.generate_quote()  # Перегенерация с учётом фильтров

if __name__ == "__main__":
    root = tk.Tk()
    app = QuoteGenerator(root)
    root.mainloop()
