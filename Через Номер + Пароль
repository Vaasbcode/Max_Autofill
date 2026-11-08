import asyncio
from datetime import datetime, timedelta, timezone
from pymax import Client  # Используем обычный Client для входа по номеру (TCP)

# ⚠️ НАСТРОЙКИ АККАУНТА И ЧАТА
PHONE = "+799900000"  # Замените на ваш номер телефона
MAX_PASSWORD = "password"  # Замените на пароль от аккаунта MAX (если 2FA включена)
TARGET_CHAT_ID = -1 # Можно узнать через web.max.ru зайдя в группу/собеседника.
MESSAGE_TEXT = "+"

# Часовой пояс (UTC+4)
SAMARA_TZ = timezone(timedelta(hours=4))

# Список праздничных дней (2024-2025)
HOLIDAYS = {
    "2024-01-01", "2024-01-02", "2024-01-03", "2024-01-04", "2024-01-05", "2024-01-06", "2024-01-07", "2024-01-08",
    "2024-02-23", "2024-03-08", "2024-04-29", "2024-04-30", "2024-05-01", "2024-05-09", "2024-05-10",
    "2024-06-12", "2024-11-04", "2024-12-30", "2024-12-31",
    "2025-01-01", "2025-01-02", "2025-01-03", "2025-01-06", "2025-01-07", "2025-01-08",
    "2025-02-24", "2025-03-10", "2025-05-01", "2025-05-02", "2025-05-08", "2025-05-09",
    "2025-06-12", "2025-06-13", "2025-11-03", "2025-11-04", "2025-12-31"
}


# Провайдер пароля для 2FA
class MyPasswordProvider:
    async def get_password(self, hint: str | None = None) -> str:
        print(f"🔐 Сервер запрашивает пароль 2FA. Отправляем '{MAX_PASSWORD}'...")
        return MAX_PASSWORD


# Провайдер для ввода SMS-кода (если потребуется)
class ConsoleSmsProvider:
    async def get_code(self, phone: str) -> str:
        code = input(f"📩 Введите SMS-код для {phone}: ")
        return code.strip()


async def main():
    print("🚀 Запуск клиента...")

    client = Client(
        phone=PHONE,
        work_dir="cache",
        session_name="main.db",
        password_provider=MyPasswordProvider(),
        # Если библиотека запросит SMS, он попросит ввести его в консоль:
        # sms_provider=ConsoleSmsProvider()
    )

    @client.on_start()
    async def on_start(client: Client):
        print("✅ Авторизация прошла успешно!")

        # Начинаем с завтрашнего дня по времени Самары
        current_date = datetime.now(SAMARA_TZ) + timedelta(days=1)
        scheduled_count = 0

        # Ограничиваем планирование на 2 месяца (60 дней) вперед
        end_date = datetime.now(SAMARA_TZ) + timedelta(days=60)

        print(
            f"📅 Планирование с {current_date.strftime('%d.%m.%Y')} по {end_date.strftime('%d.%m.%Y')} (Самара, UTC+4)...")

        while current_date <= end_date:
            date_str = current_date.strftime("%Y-%m-%d")

            # Проверяем: будний день и не праздник
            is_weekday = current_date.weekday() < 5
            is_holiday = date_str in HOLIDAYS

            if is_weekday and not is_holiday:
                # Устанавливаем время 08:30 утра по Самаре
                send_time = current_date.replace(hour=8, minute=30, second=0, microsecond=0, tzinfo=SAMARA_TZ)

                try:
                    await client.send_message(
                        chat_id=TARGET_CHAT_ID,
                        text=MESSAGE_TEXT,
                        send_at=send_time,
                        notify=True
                    )
                    scheduled_count += 1
                    print(f"[{scheduled_count}] Запланировано на {send_time.strftime('%d.%m.%Y %H:%M')} (Самара)")
                except Exception as e:
                    print(f"❌ Ошибка на {send_time.strftime('%d.%m.%Y')}: {e}")

                # Пауза 1 секунда между запросами, чтобы сервер не выдал ошибку
                await asyncio.sleep(1.0)

            current_date += timedelta(days=1)

        print(f"\n🎉 Готово! Успешно заполнено {scheduled_count} рабочих дней.")
        await client.close()

    try:
        await client.start()
    except Exception as e:
        print(f"❌ Ошибка при запуске/авторизации: {e}")


if __name__ == "__main__":
    asyncio.run(main())
