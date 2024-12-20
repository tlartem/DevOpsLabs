# Отчет. Лаба 1. Nginx

**Авторство: Цырульников Артём Алексеевич К3239**

## Задание:

1. Должен работать по https c сертификатом
2. Настроить принудительное перенаправление HTTP-запросов (порт 80) на HTTPS (порт 443) для обеспечения безопасного соединения.
3. Использовать alias для создания псевдонимов путей к файлам или каталогам на сервере.
4. Настроить виртуальные хосты для обслуживания нескольких доменных имен на одном сервере.
5. Что угодно еще под требования проекта 
*(P.S. проект лапочка он ничего не требует)*

## Зажигание

BTW I Use Arch, но в  этот раз все же поставлю Ubuntu Server на виртуалку чтобы не мусорить в системе.

![1](media/1.png)

И конечно же мущинский `neofetch`.

![2](media/2.png)

Дальше поставим пакетики.

```bash
sudo apt update
sudo apt install nginx -y
```

Запустим этого тигра, лееее:

```bash
sudo systemctl start nginx
# Ставим на автозапуск этот видеорегистратор
sudo systemctl enable nginx
```
![3](media/3.png)

Включим сетевой мост для нашего тигра, чтобы с основной системы зайти в браузер и проверить работу.

![4](media/4.png)

Ура, братья, мы завелись! (Nginx теперь тоже поддерживает брюнеток и гламурный темный режим)

## Пристегиваемся и настраиваем сиденье

### В первом пункте от нас хотят соединение *https* и настроенные сертификаты. (А еще захватим перенаправление с *http*)

Купим у файла `/etc/hosts` по братски пару доменов:

![5](media/5.jpg)

Мы обманули наш компьютер, теперь он думает, что мы смогли позволить себе доменные имена и зарегистрировались в DNS. Теперь наши тигры имеют имена, и можем обращаться по доменам.

Для обеспечения соединения по *HTTPS* нам нужно получить сертификаты. Взаимодействовать с **CloudFlare** мы не хотим, поэтому просто подпишем их сами.

Создадим каталоги для сертификатов, чтобы все хранилось по порядку:

```bash
sudo mkdir -p /etc/nginx/ssl/tigr1
sudo mkdir -p /etc/nginx/ssl/tigr2
```

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
 -keyout /etc/nginx/ssl/tigr1/tigr1.krasava.key \
 -out /etc/nginx/ssl/tigr1/tigr1.krasava.crt \
 -subj "/C=US/ST=State/L=City/O=Organization/OU=Department/CN=example.com"
```

Создаем конфиг файл для первого тигра:

`/etc/nginx/sites-available/tigr1.krasava.conf`

![6](media/6.png)

**Тут чуть-чуть подкрался `alias` я просто забыл заранее скрин сделать**

**Для второго мы просто циферку меняем, так что я не буду дублироваться!**

Создаем симлинки, чтобы Nginx знал, где тигры зимуют:

```bash
sudo ln -s /etc/nginx/sites-available/tigr1.krasava.conf /etc/nginx/sites-enabled/
```

Создадим какую нибудь хтмэльку(HTML) заглушку:

```bash
sudo mkdir -p /var/www/project1/static
sudo nano /var/www/project1/index.html
```

Как настоящая фанатка музыки, я не смог устоять:

(Во втором файле — Metallica)
![7](media/7.png)

Проверим, что нигде не накосячили:

```bash
sudo nginx -t
```

![8](media/8.png)

Слава бобер крупа! Перезапускаем Nginx и проверяем:

```bash
sudo systemctl restart nginx
```

![9](media/9.gif)

Мы достигли цели первого пункта, а второй выполнили заодно. У нас работают оба проекта, а также принудительно перенаправляются на HTTPS.

Правда, браузер не дурак и видит, что мы сами подписали сертификаты, поэтому соединение отображается красным цветом. :( Токсичный вайб.

### Настроим `alias` для удобного доступа к файлам на сервере

Добавим в наши конфиги данные строчки:

```nginx
location /static/ {
    alias /var/www/project1/static/;
}
```

Теперь все наши картинки и видео, которые находятся в папке проектов, можно получить, добавив к домену `tigr.krasava/static/{file.ex}`.

Теперь с помощью `wget` скачиваем какие-нибудь картинки с Всемирной глобальной межгалактической сверхскоростной паутинообразной цифровой инфо передающей связи через эфирное пространство с использованием электромагнитных волн для поддержания беспрерывного онлайн-доступа к безграничному океану данных и мемов.

```bash
sudo wget /var/www/project1/static/acdc.png {long hyperlink}
```

![](media/10.png)

Добавим логотип Metallica во второй проект, и можно тестировать нашу красоту.

`https://tigr1.krasava/static/acdc.png`
![](media/11.png)

`https://tigr2.krasava/static/metallica.jpg`
![](media/12.png)

## Вывод

Собсна, мы все поставили:

1. Два разных проекта хостятся на сервере с одним физ. IP. 

2. Забанили всех, кто пытается использователь непровославное *http* соединение, заставили всех использовать *https*.

3. Настроили удобный доступ к статике через `alias`.

Всем спасибо за внимание! Живите кайфуте!
