import asyncio
import json
import os
from aiogram import Bot, Dispatcher, types
from aiogram.types import Message, InlineKeyboardMarkup, InlineKeyboardButton
from aiogram.filters import Command

# Твій API Token
API_TOKEN = "7708939526:AAE7CpllzvBYC58aahcXodVI6McDt87kxaY"

# Створюємо об'єкти бота і диспетчера
bot = Bot(token=API_TOKEN)
dp = Dispatcher()

# Шлях до файлів для збереження списків
MOVIES_FILE = "movies.json"
WATCHED_FILE = "watched_movies.json"

# Завантаження даних з файлів
def load_data():
    try:
        if os.path.exists(MOVIES_FILE):
            with open(MOVIES_FILE, "r", encoding="utf-8") as f:
                movies = json.load(f)
                if not isinstance(movies, list):  # Перевірка, чи це список
                    movies = []
        else:
            movies = []

        if os.path.exists(WATCHED_FILE):
            with open(WATCHED_FILE, "r", encoding="utf-8") as f:
                watched_movies = json.load(f)
                if not isinstance(watched_movies, list):  # Перевірка, чи це список
                    watched_movies = []
        else:
            watched_movies = []

        return movies, watched_movies
    except (FileNotFoundError, json.JSONDecodeError) as e:
        print(f"Error loading data: {e}")
        return [], []

# Збереження даних у файли
def save_data(movies, watched_movies):
    try:
        with open(MOVIES_FILE, "w", encoding="utf-8") as f:
            json.dump(movies, f, ensure_ascii=False, indent=4)

        with open(WATCHED_FILE, "w", encoding="utf-8") as f:
            json.dump(watched_movies, f, ensure_ascii=False, indent=4)
    except Exception as e:
        print(f"Error saving data: {e}")

# Завантаження даних при запуску
movies, watched_movies = load_data()

# 📌 Головне меню кнопок
def main_menu():
    keyboard = InlineKeyboardMarkup(inline_keyboard=[
        [InlineKeyboardButton(text="🎬 Показати список", callback_data="show_list")],
        [InlineKeyboardButton(text="👀 Що ми бачили?", callback_data="watched_list")],
    ])
    return keyboard

# ✅ Додати фільм
@dp.message(lambda message: message.text.lower().startswith("пушок додай фільм "))
async def add_movie(message: Message):
    movie_name = message.text[len("пушок додай фільм "):].strip()
    if movie_name:
        # Перевірка на дублікати
        if movie_name not in movies:
            movies.append(movie_name)
            save_data(movies, watched_movies)  # Зберігаємо дані після додавання
            await message.answer(f"✅ Фільм **{movie_name}** додано!", reply_markup=main_menu())
        else:
            await message.answer("⚠️ Цей фільм вже є в списку!")
    else:
        await message.answer("⚠️ Напиши назву фільму після команди.")

# 📜 Показати список фільмів
@dp.message(lambda message: message.text.lower() in ["пушок список"])
async def show_movies(message: Message):
    if movies:
        movie_list = "\n".join(f"🎬 {movie}" for movie in movies)
        await message.answer(f"📜 Список фільмів:\n{movie_list}")
    else:
        await message.answer("📭 Список фільмів порожній!")

# 👀 Перемістити фільм у переглянуті
@dp.message(lambda message: message.text.lower().startswith("пушок переглянули "))
async def mark_watched(message: Message):
    movie_name = message.text[len("пушок переглянули "):].strip()
    if movie_name in movies:
        movies.remove(movie_name)
        watched_movies.append(movie_name)
        save_data(movies, watched_movies)  # Зберігаємо дані після зміни
        await message.answer(f"👀 Фільм **{movie_name}** позначено як переглянутий!")
    else:
        await message.answer("⚠️ Цього фільму немає у списку!")

# 🎥 Показати переглянуті фільми
@dp.message(lambda message: message.text.lower() in ["пушок що ми бачили"])
async def show_watched(message: Message):
    if watched_movies:
        watched_list = "\n".join(f"✅ {movie}" for movie in watched_movies)
        await message.answer(f"👀 Переглянуті фільми:\n{watched_list}")
    else:
        await message.answer("📭 Список переглянутих фільмів порожній!")

# 🗑 Видалити фільм
@dp.message(lambda message: message.text.lower().startswith("пушок удали "))
async def delete_movie(message: Message):
    movie_name = message.text[len("пушок удали "):].strip()
    if movie_name in movies:
        movies.remove(movie_name)
        save_data(movies, watched_movies)  # Зберігаємо дані після видалення
        await message.answer(f"🗑 Фільм **{movie_name}** видалено!")
    elif movie_name in watched_movies:
        watched_movies.remove(movie_name)
        save_data(movies, watched_movies)  # Зберігаємо дані після видалення
        await message.answer(f"🗑 Фільм **{movie_name}** видалено з переглянутих!")
    else:
        await message.answer("⚠️ Цього фільму немає у списку!")

# ℹ️ Показати команди
@dp.message(lambda message: message.text.lower() in ["пушок шо вмієш"])
async def show_commands(message: Message):
    commands = """
📌 **Доступні команди:**
- **Пушок додай фільм [назва]** – додати фільм
- **Пушок список** – показати список фільмів
- **Пушок переглянули [назва]** – позначити як переглянутий
- **Пушок що ми бачили** – список переглянутих
- **Пушок удали [назва]** – видалити фільм
- **Пушок шо вмієш** – список команд
    """
    await message.answer(commands)

# 📍 Обробка кнопок
@dp.callback_query(lambda c: c.data == "show_list")
async def process_show_list(callback_query: types.CallbackQuery):
    await show_movies(callback_query.message)

@dp.callback_query(lambda c: c.data == "watched_list")
async def process_watched_list(callback_query: types.CallbackQuery):
    await show_watched(callback_query.message)

# 🚀 Запуск бота
async def main():
    try:
        await dp.start_polling(bot)
    except Exception as e:
        print(f"Bot error: {e}")

if __name__ == "__main__":
    asyncio.run(main())
