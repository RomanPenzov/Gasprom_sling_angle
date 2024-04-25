# Gasprom_sling_angle
<center> <img src = https://stavropol.termaxpanel.ru/wa-data/public/photos/79/09/979/979.1080.png alt="drawing" style="width:400px;">
<center> <img src = https://static.tildacdn.com/tild3930-3834-4530-a365-656331396337/mfti.png alt="drawing" style="width:400px;">

## Цель данного проекта - детекция строп и угла между ними при погрузочных-разгрузочных работах 

## 0. Загрузка баз данных для дальнейшей работы
* Здесь можно скачать [видео от заказчика](https://drive.google.com/file/d/1snD7mVoE1mqoifNw-IFdgtk39iNDrh2B/view?usp=sharing "видео от заказчика")

## 1. Этап предобработки базы
* в редакторе VS Code открываем новый ноутбук под именем sling_angle.ipynb
* полученный от заказчика видеофайл CNTRL.mp4 длится около 5 минут и имеет около 7500 кадров. Мы имеем ограниченные ресурсы, поэтому, используя библиотеку cv2, берем из этого видео каждый 75-й кадр и получаем укороченный видеофайл sampled_video.mp4
* делаем раскадровку этого укороченного видео и получаем набор изображений из 100 кадров. Сохраняем каждый кадр из этого нового видео "sampled_video.mp4" в отдельный файл JPG. Каждое изображение сохраняется в папке "frames" под именем вида "frame_0.jpg", "frame_1.jpg", и так далее. Этот набор изображений можно будет использовать для дальнейшего аннотирования (разметки строп на изображениях).
* визуально проматриваем эти кадры на предмет их пригодности для детекции строп и определения угла между ними (осталось более 50%).
* делаем аннотирование (отметить СТРОПЫ на кадрах) ранее сохраненных кадров на вебплатформе Roboflow для получения базы data.yaml . Для увеличения этой базы применяем аугментацию (изменения в размерах, насыщенности, яркости, смещения, ...), что существенно увеличивает нашу базу (в 2.5 раза). В итоге получаем архив с анотированной базой изображений [аннотированные изображения](https://drive.google.com/file/d/1slw0ZA8pRTDywD_s0uub0SC4xit6VlVu/view?usp=sharing "аннотированные изображения")

## 2. Дообучение модели YOLOv8 на данных заказчика
* используем современную обученную модель YOLOv8 из библиотеки ultralytics для получения в итоге модели, которая будет делать детекцию строп на видео
* так как мы ограничены в ресурсах, то используем бесплатные технические ресурсы компании Google на Colab (в течение суток предоставляется возможность испольховать GPU с большой оперативной памятью) - это сильно сокращает время обучения (мой компьютер просто зависал, не дойдя обучение до конца). Вот наш название Gasprom_train-yolov8-keypoint_24042024.ipynb файла ноутбук на Colab [ноутбук обучения по стропам](https://colab.research.google.com/drive/1BsBdWl90nMd1-s8Gt7CdCm8ZKIIJj6Dj?usp=sharing "ноутбук обучения по стропам") 
* запускаем эту базовую модель YOLOv8 обучаться на нашей аннотированной базе на 100 эпохах и получаем высокую точность (более 90%) детекции самих строп и их классификации как объектов. Мы понимаем, что идеально увеличить базу изображений хотя бы до 1000. Обучение на 100 эпохах длилось 0.166 часа. В итоге мы получили веса для нашей модели по стопам, чтобы в дальнейшем их ипользовать без предварительного обучения модели - вот файл best.pt (переименовываем в best_keypoint_24042024.pt) с этими весами [веса для модели по стропам](https://drive.google.com/file/d/1LRT1B9FksO6UWzWXmGMA5gbzQgnxqt3f/view?usp=sharing "веса для модели по стропам")
* тут же делаем проверку этой обученной модели на нескольких изображениях из видео, где получаем положительные численные и визуальные результаты (точность 85% и рамка детекции нарисована верно)

## 3. Сам процесс детекции и определения угла между стропами на видео заказчика
* делаем допущение по геометрии - определяем угол между строп в проекции 2D (безопасность предела допустимого угла определяется тем, что этот угол в проекции больше оригинального угла в 3D, т.е. будем иметь некоторый запас в пределах 10%). Иначе нужно было аннотировать отдельно концы каждой стропы (заложить трудозатраты) - результат не сильно бы изменился.
* На каждом кадре видео делаем детекцию объектов строп и рисуем рамки вокруг них (с координатами, вероятностью и величиной угла между стропами). На основании координат рамки можем определить угол между стропами: верхняя точка - середина верхнего ребра рамки, и две нижних точки - концы нижнего ребра рамки.
* В вероятность детекции выводим сверху рамки синим цветом. Значение угла выводим снизу рамки красным цветом. Максимальный угол выводим на печать.
* Далее получаем новое видео уже с этой детекцией.
</left>

## 4. Технологии и средства проекта
* VS Code
* Python
* cv2
* pandas
* numpy
* ultralytics YOLOv8
* Roboflow
* PIL
* google.colab
* google.drive
* zipfile

## 5. Характерное для проекта (проблемы, с которыми пришлось столкнуться, уникальные составляющие проекта)
* видеофайлы имеют 30 кадров в 1 секунду, что избыточно - можно оставлять для обработки 1 кадр на 1 секунду, но тогда в итоговом обработанном видео надо будет дублировать каждый кадр несколько раз, чтобы скорость видео была приемлема для просмотра. Для этого можно предусмотреть два парметра: один - сколько оставлять кадров в первоначальном видео (каждый 30, 75 или ...), второй - количество для дублирования кадров (возможно, что вернуть теже 30 кадров или меньше).
* много работы по аннотированию изображений
* нужны хорошие ресурсы для обучение новой модели (ограниченные возможности GPU на Colab от Google)

## 6. Заключение
Используя данный проект, можно быстро получать анализ безопасного угла между стропами при погрузочно-разгрузочных работах

## 7. Внедрение в производство
надо реализовать два способа использования этой модели по определению угла между стропами:
* в телеграм
* сайт с разными видами сервиса для разных технологических операций (в том числе разгрузка-погрузка) - удобно сделать на Django

## 8. Возможности улучшений
* на концах строп могут быть некоторые геометричекие объекты разного цвета (маленькие пластиковые шары или цилиндрики), что улучшит детекцию и дифференциацию нужных точек.

## 9. Как использовать.
* загрузить файл sling_angle.ipynb (для простоты на Colab)
* в части 4 в переменной video_path прописать название своего видеофайла и загрузить этот файл в Colab
* запустить на исполнение часть 4 и в итоге получим новый видеофайл output.mp4 с рамками детекции и со значениями углов между стропами
* есть вариант для веба на Flask (требуется настройка - загрузка файла для обработки и выгрузка обработанного файла)

## 10. Контакты
e-mail: romanpenzov@yandex.ru
телефон: +7(916)193-93-56