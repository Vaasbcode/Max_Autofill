import asyncio
from datetime import datetime, timedelta, timezone
from pymax import WebClient

# ⚠️ Настройки
TARGET_CHAT_ID = -1 # Номер чата/группы/собеседника, можно найти в версии web.max.ru
MESSAGE_TEXT = "+"

# Часовой пояс (UTC+4)
SAMARA_TZ = timezone(timedelta(hours=4))

# Список праздничных дней (на 2024-2025 годы)
HOLIDAYS = {
    "2024-01-01", "2024-01-02", "2024-01-03", "2024-01-04", "2024-01-05", "2024-01-06", "2024-01-07", "2024-01-08",
    "2024-02-23", "2024-03-08", "2024-04-29", "2024-04-30", "2024-05-01", "2024-05-09", "2024-05-10",
    "2024-06-12", "2024-11-04", "2024-12-30", "2024-12-31",
    "2025-01-01", "2025-01-02", "2025-01-03", "2025-01-06", "2025-01-07", "2025-01-08",
    "2025-02-24", "2025-03-10", "2025-05-01", "2025-05-02", "2025-05-08", "2025-05-09",
    "2025-06-12", "2025-06-13", "2025-11-03", "2025-11-04", "2025-12-31"
}


class PrintQrUrl:
    async def show_qr(self, qr_url: str):
        print("\n" + "=" * 60)
        print("📱 ОТСКАНИРУЙТЕ QR-КОД В ПРИЛОЖЕНИИ MAX")
        print(f"Ссылка для генерации QR: {qr_url}")
        print("=" * 60 + "\n")


async def main():
    client = WebClient(
        work_dir="cache",
        session_name="web.db",
        qr_provider=PrintQrUrl(),
    )

    @client.on_start()
    async def on_start(client: WebClient):
        print("✅ Авторизация прошла успешно!")

        # Начинаем с завтрашнего дня по времени Самары
        current_date = datetime.now(SAMARA_TZ) + timedelta(days=1)
        scheduled_count = 0

        # Рассчитываем дату окончания: 2 месяца (60 дней) от текущей даты
        end_date = datetime.now(SAMARA_TZ) + timedelta(days=60)

        print(
            f"📅 Планирование с {current_date.strftime('%d.%m.%Y')} по {end_date.strftime('%d.%m.%Y')} (Самара, UTC+4)...")

        while current_date <= end_date:
            date_str = current_date.strftime("%Y-%m-%d")

            # Проверяем: будний день и не праздник
            is_weekday = current_date.weekday() < 5
            is_holiday = date_str in HOLIDAYS

            if is_weekday and not is_holiday:
                # Устанавливаем время 08:10 утра по Самаре
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

                # Пауза, чтобы не забанили за флуд запросами
                await asyncio.sleep(0.5)

            current_date += timedelta(days=1)

        print(f"\n🎉 Готово! Успешно заполнено {scheduled_count} рабочих дней.")
        await client.close()

    print("🚀 Запуск клиента. Ожидание сканирования QR-кода...")
    await client.start()


if __name__ == "__main__":
    asyncio.run(main())
