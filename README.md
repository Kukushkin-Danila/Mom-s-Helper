# Mom-s-Helper

https://t.me/mom_helper_bot

## Аннотация

Суть проекта заключается в распознавании символов по уходу за одеждой. Реализация проекта делится на две основные части: распознавание символов по фотографии с помощью Detectron2 и в режиме реального времени с помощью YOLO5. 

## Сбор данных и препроцессинг

Для нашего проекта мы собрали ~700 фотографий лейблов и на каждый из них нанесли bounding boxes. Всего у нас получилось 35 классов. На самом деле, значков существует чуть больше, но некоторые очень редко встречаются или относятся к определенному производителю одежды. Так же, мы столкнулись с тем, что символы не имеют единого требования к оформлению. Например, для азиатских регионов некоторые символы сильно отличаются и на них нанесены иероглифы. Такие знаки мы не размечали.
Далее для наших фотографий мы сделали аугментацию и, в итоге, у нас получился датасет в ~2700 фотографий. 

## Обучение для изображений

Для задачи object detection мы использовали предобученную на датасете COCO модель faster_rcnn_X_101_32x8d_FPN_3x. Мы выбрали именно ее, так как она имеет наибольшое колчиство параметров и по метрикам качества boxAP имеет наивысшее значение среди предобученных моделей из "зоопарка моделей" Detectron2. На сайте paperwithcode можно увидеть, что данная модель на самом деле является лучшей. Это было выяснено и эмпирических путем: мы экспереминтировали с разными моделями, пробовали Fast RCNN и RetinaNet, но их результаты были хуже. Все обучение проходило в Google Colab на их видеокартах. Оценивали модели мы по метрике AP50.

## Обучение для Real-Time Detection 

Далее мы решили испытать нашу модель в режиме реального времени. Для начала мы точно так же использовали Faster-RCNN из Detectrion'a 2, но возникли некоторые трудности: из-за большого количества параметров и нехватки наших мощностей real-time работал очень медленно, мы решили применить одноэтапный метод детекции объектов - архитектуру YOLO5. 17 часов мы обучали самую маленькую модель YOLO, далее мы использовали скрипт с сайта YOLO для режима реального времени и подгрузили туда модель. Оценивали модель мы с помощью метрики mAP, наш результат 73%. 

## Имплементация в telegram бота

Для реализации проекта мы использовали сервис Telegram. Делали мы то с помощью библиотеки aiogram — ассинхронного фреймворка для Telegram Bot API основанный на asyncio и aiohttp.

# Структура бота 

**bot.py**

Данный модуль является основным для самого бота, так там реализованы стартовое приветственное сообщение, проверка на тип вложения (боту можно отправлять только фотографию, иначе - ошибка), а так же выдача обработанной фотографии и текст-инструкция по уходу за одеждой. Текст разделен "смысловыми" эмодзи для более упрощенного восприятия информации. 

**preditior.py**

После того, как пользователь загрузил фотографию в бот она попадает в этот модуль. Здесь она обрабатывается с помощью нашей обученной модели через веса, которые мы сохранили ранее из Google Colab. Здесь распознаются значки и на их границы наносятся bounding boxes. Каждый символ относится к определенному классу, которые пронумерованы. Далее этот номер класса соотносится со словарем Несмотря на то, что размеченные символы на bounding boxes могут повторяться из-за разных географических регионов, описания к классам никогда не повторяются.

**static_text.py**

Здесь находится текста для нашего бота, начиная от приветственного сообщения, заканчивая описаниями классов в виде словаря. 

**requirements.txt**

В этом файле лежат библиотеки, которые мы использовали и которые необходимо установить локально на компьютер, чтобы запустить бота. 
