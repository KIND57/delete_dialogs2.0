from telethon import TelegramClient, errors
import asyncio

async def main():
    print("Чтобы получить api_id и api_hash, зайдите на https://my.telegram.org и войдите под своим номером телефона.")
    print("Там в разделе 'API development tools' создайте новое приложение и получите эти данные.")

    # Запрос данных у пользователя
    api_id = int(input("Введите ваш api_id (число): "))
    api_hash = input("Введите ваш api_hash: ")
    phone = input("Введите ваш номер телефона в международном формате (например +1234567890): ")
    delete_type = input("Выберите тип удаления ('all', 'channels', 'dialogs'): ").strip().lower()
    if delete_type not in ['all', 'channels', 'dialogs']:
        print("Некорректный выбор, устанавливаю по умолчанию 'all'.")
        delete_type = 'all'

    # Создаем клиента
    client = TelegramClient('session', api_id, api_hash)

    # Входим в аккаунт
    await client.start(phone=phone)
    print('Успешно вошли в аккаунт.')

    # Получение списка диалогов
    dialogs = await client.get_dialogs()
    print(f'Найдено {len(dialogs)} чатов/каналов.')

    # Обработка каждого диалога
    for dialog in dialogs:
        entity = dialog.entity
        title = getattr(entity, 'title', 'чат')
        # Определяем, нужно ли удалять исходя из типа
        if delete_type == 'all':
            should_delete = True
        elif delete_type == 'channels' and getattr(entity, 'broadcast', False):
            should_delete = True
        elif delete_type == 'dialogs' and not getattr(entity, 'broadcast', False):
            should_delete = True
        else:
            should_delete = False

        if should_delete:
            print(f'Удаляю: {title}')
            try:
                await client.delete_dialog(entity)
            except errors.FloodWaitError as e:
                print(f'Лимит запросов, ждем {e.seconds} секунд.')
                await asyncio.sleep(e.seconds)
                await client.delete_dialog(entity)
            except Exception as e:
                print(f'Ошибка при удалении "{title}": {e}')

    print('Удаление завершено.')

if __name__ == '__main__':
    asyncio.run(main())
