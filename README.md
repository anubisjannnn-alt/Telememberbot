import os
import requests
from aiogram import Bot, Dispatcher, types, executor
from dotenv import load_dotenv

# Cargar variables de .env
load_dotenv()
TELEGRAM_TOKEN = os.getenv("TELEGRAM_TOKEN")
COINBASE_API_KEY = os.getenv("COINBASE_API_KEY")
PROVIDER_API_KEY = os.getenv("PROVIDER_API_KEY")

bot = Bot(token=TELEGRAM_TOKEN)
dp = Dispatcher(bot)

# Catálogo de servicios legales
CATALOG = {
    "1": {"title": "Web Traffic", "service_id": 5, "price": 5},
    "2": {"title": "SEO Package", "service_id": 6, "price": 10}
}

@dp.message_handler(commands=["start"])
async def start(message: types.Message):
    text = "Hola! Elige un servicio:\n"
    for k,v in CATALOG.items():
        text += f"{k}. {v['title']} — ${v['price']}\n"
    text += "Responde con el número del servicio que quieres."
    await message.reply(text)

@dp.message_handler(lambda m: m.text and m.text in CATALOG)
async def handle_choice(message: types.Message):
    choice = message.text
    item = CATALOG[choice]
    
    # Crear invoice en Coinbase Commerce
    url = "https://api.commerce.coinbase.com/charges"
    headers = {"X-CC-Api-Key": COINBASE_API_KEY, "X-CC-Version": "2018-03-22"}
    data = {
        "name": item["title"],
        "pricing_type": "fixed_price",
        "local_price": {"amount": str(item["price"]), "currency": "USD"}
    }
    r = requests.post(url, json=data, headers=headers).json()
    pay_url = r["data"]["hosted_url"]
    
    await message.reply(f"Pago aquí: {pay_url}\nCuando se confirme, te entregaré el servicio automáticamente.")
