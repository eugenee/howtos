# OpenWEBUI+OpenAI

Created: March 24, 2025 7:52 PM
Tags: chatgpt, deepseek, docker, howto, litellm, ollama, openai, openwebui

Як не платити 20$ за платний план chatgpt? Я використовую chatgpt не сказати, що дуже часто, може 2-3 рази на тиждень, але майже завжди я натикаюсь на ліміт безкоштовного користування.

![image.png](image.png)

І двадцятка баксів, начебто і не так багато, але я точно розумію, що не використовую цей функціонал за який заплатив. От якби була б можливість платити не за місяць, а за день, а краще за годину, а ще краще за кожне повідомлення. 

І така можливість є. Потрібен тільки OpenWEBUI та обліковий запис для використання API OpenAI. 

```bash

https://openwebui.com/
```

**Open WebUI** 

Це інтерфейс для спілкування з різними мовними моделями. Сюди можна додавати як власні моделі, так і загальновідомі хмарні моделі, як chatgpt, deepseek та інші.

![demo-d3952c8561c4808c1d447fc061c71174.gif](demo-d3952c8561c4808c1d447fc061c71174.gif)

На сайті є інструкція, як запускати інтерфейс під різними системами, я обрав собі docker на linux

Для цього створюємо yaml файл

```yaml
services:
  ollama:
    image: ollama/ollama
    container_name: ollama
    restart: unless-stopped
    volumes:
      - ollama_data:/root/.ollama
    ports:
      - "11434:11434"
  openwebui:
    image: ghcr.io/open-webui/open-webui:0.5.20
    container_name: openwebui
    ports:
      - "8080:8080"
    environment:
      # Ollama Config
      - OLLAMA_BASE_URL=http://ollama:11434
    volumes:
      - data:/app/backend/data:rw
#    networks:
#      - frontend
    labels:
      - traefik.enable=true
      - traefik.http.routers.openwebui.rule=Host(`192.168.88.205`)
      - traefik.http.routers.openwebui.entrypoints=websecure
      - traefik.http.routers.openwebui.tls=true
      - traefik.http.routers.openwebui.tls.certresolver=cloudflare
      - traefik.http.routers.openwebui.service=openwebui
      - traefik.http.services.openwebui.loadBalancer.server.port=8080
    restart: unless-stopped

volumes:
  data:
    driver: local
  ollama_data:
networks:
  frontend:
    external: true
```

Після цього запускаємо наші контейнери 

```bash
docker compose up -d
```

Перший запуск буде довгим, там десь 1,5 гіга скачається

```bash
sudo docker ps
CONTAINER ID   IMAGE                                  COMMAND               CREATED         STATUS                             PORTS
   NAMES
60a78cbeca08   ghcr.io/open-webui/open-webui:0.5.20   "bash start.sh"       2 minutes ago   Up 13 seconds (health: starting)   0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp     openwebui
58d42a387fd7   ollama/ollama                          "/bin/ollama serve"   2 minutes ago   Up 13 seconds                      0.0.0.0:11434->11434/tcp, [::]:11434->11434/tcp  ollama
```

Треба ще трохи часу

Тепер відкриваємо наш інтерфейс

```bash
http://192.168.88.205:8080
```

Треба створити адміністратора

![image.png](image%201.png)

Після входу ми бачимо знайомий інтерфейс: зліва наші запити до моделей, праворуч активний діалог. В активному діалозі є можливість обрати модель, до якої ми робимо запит

І зараз нам треба додати сюди наші моделі

![image_2025-03-22_20-05-25.png](image_2025-03-22_20-05-25.png)

Відкриваємо профіль(1), заходимо в налаштування(2)

Переходимо в розділ “З’єднання”

![image_2025-03-22_20-06-20.png](image_2025-03-22_20-06-20.png)

![image_2025-03-22_20-07-02.png](image_2025-03-22_20-07-02.png)

Тепер розберемось з цінами та моделями, і зробимо наш APIключ, для цього заходимо на

```bash
[https://openai.com/api](https://openai.com/api)
```

Реєструємось, або заходимо з обліковим записом chatgpt і зліва відкриваємо розділ 
API Pricing. Тут вказано ціни за 4 моделей

![image.png](image%202.png)

![image.png](image%203.png)

отож, якщо приблизно, модель 4o по 10 баксів за один мільйон токенів. Токен, теж спрощенно, це як слово, там складна система підрахунку, але ми не про це. Головна засада тут для розробників, якщо ви просите написати код, то тут 1 токен = 1 символ коду.

Я закинув 5USD

![image.png](image%204.png)

Далі, в розділі API Keys створив перший ключ

![image_2025-03-22_20-15-12.png](image_2025-03-22_20-15-12.png)

1. Натискаємо створити
2. Обираємо назву цьому ключу
3. У мене один проект, тому тут просто дефолтне значення
4. І натискаємо Зберігання

Після чого копіюємо ключ.

![image.png](image%205.png)

 і повертаємось до OpenWEBUI. 

![image_2025-03-22_20-20-06.png](image_2025-03-22_20-20-06.png)

Тепер дивимось, які нам доступні моделі

![image_2025-03-22_20-21-35.png](image_2025-03-22_20-21-35.png)

Доречі, ми можемо обрати одразу декілька моделей і відправити один запит

![image_2025-03-22_21-00-42.png](image_2025-03-22_21-00-42.png)

Бачимо, як різні моделі відповідають на дитячу загадку.

Тепер додамо ще хмарних моделей від інших провайдерів.

```bash
https://console.groq.com
```

наприклад, тут ми можемо отримати моделі

![image_2025-03-22_21-05-50.png](image_2025-03-22_21-05-50.png)

і додамо справжнього грока

```bash
https://console.x.ai
```

А тепер те саме питання задамо на три моделі

![image.png](image%206.png)

Зліва так само є ваші чати з моделями, система самом додає до них якийсь емоджі. І є зручна штука, якої я не бачив в chatgpt: є можливість розкидати по папкам діалоги

![image.png](image%207.png)

![image.png](image%208.png)

![image.png](image%209.png)