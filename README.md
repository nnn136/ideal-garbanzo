# Training Planner (План тренировок)

**Описание программы:**
Приложение "Training Planner" предназначено для планирования тренировок, позволяет пользователям добавлять записи о тренировках, фильтровать их по типу и дате, а также сохранять и загружать данные в формате JSON. Приложение имеет интуитивно понятный интерфейс, что позволяет легко управлять записями о тренировках.

## Пошаговая инструкция

### Шаг 1: Создание интерфейса

Для создания графического интерфейса мы можем использовать библиотеку Tkinter. Пример кода для реализации интерфейса:

```python
import tkinter as tk
from tkinter import messagebox, ttk
import json
import os
from datetime import datetime

class TrainingPlanner:
    def __init__(self, master):
        self.master = master
        master.title("Training Planner")

        # Поля ввода
        self.date_label = tk.Label(master, text="Дата (гггг-мм-дд):")
        self.date_label.pack()

        self.date_entry = tk.Entry(master)
        self.date_entry.pack()

        self.type_label = tk.Label(master, text="Тип тренировки:")
        self.type_label.pack()

        self.type_entry = tk.Entry(master)
        self.type_entry.pack()

        self.duration_label = tk.Label(master, text="Длительность (минуты):")
        self.duration_label.pack()

        self.duration_entry = tk.Entry(master)
        self.duration_entry.pack()

        self.add_button = tk.Button(master, text="Добавить тренировку", command=self.add_training)
        self.add_button.pack(pady=5)

        # Таблица для отображения тренировок
        self.trainings_list = ttk.Treeview(master, columns=("date", "type", "duration"), show='headings')
        self.trainings_list.heading("date", text="Дата")
        self.trainings_list.heading("type", text="Тип тренировки")
        self.trainings_list.heading("duration", text="Длительность (мин)")
        self.trainings_list.pack(pady=10)

        # Поле для фильтрации
        self.filter_label = tk.Label(master, text="Фильтр по типу тренировки:")
        self.filter_label.pack()

        self.filter_entry = tk.Entry(master)
        self.filter_entry.pack()

        self.filter_button = tk.Button(master, text="Фильтровать", command=self.filter_trainings)
        self.filter_button.pack(pady=5)

        self.trainings = []
        self.load_trainings()

    def add_training(self):
        date_str = self.date_entry.get()
        training_type = self.type_entry.get()
        duration_str = self.duration_entry.get()

        # Проверка корректности ввода
        try:
            date = datetime.strptime(date_str, '%Y-%m-%d')
            duration = int(duration_str)
            if duration <= 0:
                raise ValueError("Длительность должна быть положительным числом.")
        except ValueError as e:
            messagebox.showerror("Ошибка", f"Некорректный ввод: {e}")
            return

        training = {
            "date": date_str,
            "type": training_type,
            "duration": duration
        }
        self.trainings.append(training)
        self.update_training_list()
        self.save_trainings()

    def update_training_list(self):
        for item in self.trainings_list.get_children():
            self.trainings_list.delete(item)
        for training in self.trainings:
            self.trainings_list.insert("", tk.END, values=(training["date"], training["type"], training["duration"]))

    def filter_trainings(self):
        training_type_filter = self.filter_entry.get().lower()
        filtered_trainings = [training for training in self.trainings if training_type_filter in training["type"].lower()]
        self.trainings_list.delete(*self.trainings_list.get_children())
        for training in filtered_trainings:
            self.trainings_list.insert("", tk.END, values=(training["date"], training["type"], training["duration"]))

    def load_trainings(self):
        if os.path.exists("trainings.json"):
            with open("trainings.json", "r") as f:
                self.trainings = json.load(f)
            self.update_training_list()

    def save_trainings(self):
        with open("trainings.json", "w") as f:
            json.dump(self.trainings, f)

if __name__ == "__main__":
    root = tk.Tk()
    app = TrainingPlanner(root)
    root.mainloop()
```

### Шаг 2: Добавление тренировки

Кнопка "Добавить тренировку" вызывает метод `add_training()`, который проверяет вводимые данные и добавляет новую тренировку в список и таблицу.

### Шаг 3: Фильтрация тренировок

Функционал фильтрации реализован в методе `filter_trainings