import tkinter as tk
from tkinter import ttk, messagebox, filedialog
import json
import os
from datetime import datetime

# --- Константы ---
DATA_FILE = "weather_diary.json"

# --- Класс приложения ---
class WeatherDiaryApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Weather Diary")
        self.records = []
        self.load_data()

        # --- Создание виджетов ---
        self.create_widgets()
        self.update_treeview()

    def create_widgets(self):
        # Рамка для ввода данных
        input_frame = ttk.LabelFrame(self.root, text="Добавить запись")
        input_frame.pack(pady=10, padx=10, fill="x")

        # Дата
        ttk.Label(input_frame, text="Дата (ДД.ММ.ГГГГ):").grid(row=0, column=0, padx=5, pady=5, sticky="e")
        self.date_entry = ttk.Entry(input_frame, width=15)
        self.date_entry.grid(row=0, column=1, padx=5, pady=5)

        # Температура
        ttk.Label(input_frame, text="Температура (°C):").grid(row=1, column=0, padx=5, pady=5, sticky="e")
        self.temp_entry = ttk.Entry(input_frame, width=15)
        self.temp_entry.grid(row=1, column=1, padx=5, pady=5)

        # Описание
        ttk.Label(input_frame, text="Описание:").grid(row=2, column=0, padx=5, pady=5, sticky="ne")
        self.desc_entry = ttk.Entry(input_frame, width=25)
        self.desc_entry.grid(row=2, column=1, padx=5, pady=5, sticky="w")

        # Осадки
        self.rain_var = tk.BooleanVar()
        ttk.Checkbutton(input_frame, text="Осадки", variable=self.rain_var).grid(row=3, column=0, columnspan=2, pady=5)

        # Кнопка добавления
        ttk.Button(input_frame, text="Добавить запись", command=self.add_record).grid(row=4, column=0, columnspan=2, pady=10)

        # Рамка для фильтрации
        filter_frame = ttk.LabelFrame(self.root, text="Фильтрация")
        filter_frame.pack(pady=10, padx=10, fill="x")

        # Фильтр по дате
        ttk.Label(filter_frame, text="Фильтр по дате:").grid(row=0, column=0, padx=5, pady=5)
        self.filter_date = ttk.Entry(filter_frame, width=15)
        self.filter_date.grid(row=0, column=1, padx=5, pady=5)

        # Фильтр по температуре (выше)
        ttk.Label(filter_frame, text="Выше (°C):").grid(row=1, column=0, padx=5, pady=5)
        self.filter_temp = ttk.Entry(filter_frame, width=15)
        self.filter_temp.grid(row=1, column=1, padx=5, pady=5)

        ttk.Button(filter_frame, text="Применить фильтр", command=self.apply_filter).grid(row=2, column=0, columnspan=2, pady=10)

        # Таблица записей
        columns = ("date", "temp", "desc", "rain")
        self.tree = ttk.Treeview(self.root, columns=columns, show="headings")
        
        self.tree.heading("date", text="Дата")
        self.tree.heading("temp", text="Температура (°C)")
        self.tree.heading("desc", text="Описание")
        self.tree.heading("rain", text="Осадки")
        
        self.tree.pack(padx=10, pady=10, fill="both", expand=True)

    def add_record(self):
        date = self.date_entry.get().strip()
        temp = self.temp_entry.get().strip()
        desc = self.desc_entry.get().strip()
        
        # Валидация ввода
        try:
            datetime.strptime(date, "%d.%m.%Y")
            if not date:
                raise ValueError("Дата не указана.")
            if not temp:
                raise ValueError("Температура не указана.")
            temp_float = float(temp)
            if not desc:
                raise ValueError("Описание не может быть пустым.")
            
            record = {
                "date": date,
                "temp": temp_float,
                "desc": desc,
                "rain": self.rain_var.get()
            }
            self.records.append(record)
            self.update_treeview()
            self.clear_entries()
            self.save_data()
            
            messagebox.showinfo("Успех", "Запись добавлена!")
            
        except ValueError as e:
            messagebox.showerror("Ошибка ввода", str(e))

    def clear_entries(self):
        self.date_entry.delete(0, tk.END)
        self.temp_entry.delete(0, tk.END)
        self.desc_entry.delete(0, tk.END)
        self.rain_var.set(False)

    def update_treeview(self):
        for i in self.tree.get_children():
            self.tree.delete(i)
        
        for rec in self.records:
            rain_text = "Да" if rec["rain"] else "Нет"
            self.tree.insert("", tk.END,
                             values=(rec["date"], rec["temp"], rec["desc"], rain_text))

    def apply_filter(self):
        filter_date = self.filter_date.get().strip()
        filter_temp_str = self.filter_temp.get().strip()
        
        filtered = self.records.copy()
        
        if filter_date:
            try:
                datetime.strptime(filter_date, "%d.%m.%Y")
                filtered = [r for r in filtered if r["date"] == filter_date]
            except ValueError:
                messagebox.showerror("Ошибка", "Неверный формат даты для фильтра.")
                return

        if filter_temp_str:
            try:
                temp_val = float(filter_temp_str)
                filtered = [r for r in filtered if r["temp"] > temp_val]
            except ValueError:
                messagebox.showerror("Ошибка", "Температура фильтра должна быть числом.")
                return

        for i in self.tree.get_children():
            self.tree.delete(i)
        
        for rec in filtered:
            rain_text = "Да" if rec["rain"] else "Нет"
            self.tree.insert("", tk.END,
                             values=(rec["date"], rec["temp"], rec["desc"], rain_text))

    def save_data(self):
        try:
            with open(DATA_FILE, 'w', encoding='utf-8') as f:
                json.dump(self.records, f, ensure_ascii=False, indent=4)
            messagebox.showinfo("Сохранение", "Данные успешно сохранены в JSON.")
        except Exception as e:
            messagebox.showerror("Ошибка сохранения", str(e))

    def load_data(self):
        if os.path.exists(DATA_FILE):
            try:
                with open(DATA_FILE, 'r', encoding='utf-8') as f:
                    self.records = json.load(f)
                    # Конвертация температуры из строки в число при необходимости
                    for r in self.records:
                        if isinstance(r.get("temp"), str):
                            r["temp"] = float(r["temp"])
            except Exception as e:
                messagebox.showerror("Ошибка загрузки", f"Не удалось загрузить данные: {e}")
                self.records = []

# --- Запуск приложения ---
if __name__ == "__main__":
    root = tk.Tk()
    app = WeatherDiaryApp(root)
    root.mainloop()
