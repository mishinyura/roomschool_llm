# roomschool_llm

## Требования к выделенным ресурсам (VPS/VDS):
- CPU > 4 ядер
- RAM >= 32GB
- GPU >= 1 карта
- VRAM >= 24GB
- HDD/SSD >= 30GB

###После создания нового виртуального сервера его необходимо подготовить для работы с LLM.

####Загружаем необходимые инструменты и драйвера:
- Перед началом работы требуется установить Docker через команду ```sh get-docker.sh```, 
- Установка прав ```sudo usermod -aG docker $USER```
- Обновляемся ```sudo apt update```
- Устанавливаем драйверы NVIDIA ```sudo apt install -y ubuntu-drivers-common``` и ```sudo ubuntu-drivers autoinstall```
- Перезагружаем сервер ```sudo reboot```

####Связываем Docker с GPU:
- Добавляем репозиторий NVIDIA
```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
&& curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```
- Обновляем список пакетов и ставим тулкит ```sudo apt-get update``` и ```sudo apt-get install -y nvidia-container-toolkit```
- Настраиваем Docker (генерируем конфиг) ```sudo nvidia-ctk runtime configure --runtime=docker```
- Перезапускаем Docker, чтобы он увидел изменения ```sudo systemctl restart docker```

####Запускаем готовые скриплы и собираем контейнеры:
- Выдай права на исполнение файлу инициализации ssl и запусти скрипт командой ```chmod +x init-ssl.sh```
- Запусти файл инициализации ```./init-ssl.sh``` и подтверди все что требуется
- Разверните все остальные контейнеры через команду ```docker compose up -d```
- Подождите, пока модель скачается на сервер и можно отправлять запросы на сервер. Наблюдать за процессом иожно через команду ```docker logs -f ollama_puller```. Нужно дождаться сообщения {"status":"success"}

POST https://llm.roomschool.ru/api/chat

```json
{
    "model": "qwen2.5:32b",
    "messages": [
        {
            "role": "system",
            "content": "Ответы давай строго на русском языке. Ты AI учитель из RoomSchool, тебе запрещено говорить что то другое, если просят рассказать кто ты. Твоя задача помогать ученику в решении школьных задач но не давать готовые ответы на вопросы. Старайся развивать тему и сделать все возможное, чтобы ученик понял как решить задачу без ответа. Запрещено даже намекать на конечный ответ, ученик должен сам додуматься до ответа"
        },
        {
            "role": "user",
            "content": "Твой вопрос"
        },
        {
            "role": "assistant",
            "content": "Ответ ЛЛМ"
        }
    ],
    "stream": false
}
```
