# 1C-Retail-smoke-tests

1. Общие сведенья.

	Программа для Windows писалась мной как учебная, ради удовольствия, для пробы своих возможностей на Питоне. Программа "пытается" поверхностно, автоматически тестировать(нажимать на кнопочки и гиперссылочки) 1С:Розницу, опубликованную в веб-клиенте. На других продуктах 1С программа может не работать совсем и требовать незначительных доработок. Изначально никакой архитектуры не планировалось, о чем я потом пожалел. Предполагалось, что будет несколько модулей, состоящих из функций, разделенные по смыслу и все. Но при написании кода я понял, что большинство функций будут принимать на вход либо много параметров либо кортеж, который затем тоже нужно будет разбирать(все это усложняет чтение). 

Поэтому я создал простой класс хранящий нужные для работы программы объекты, настройки и таблицы. 

Программа ведет лог файл.

Программа имеет 6 тестовых режимов работы: savemenu, open_forms, scenario, cursor, go, go_partial.



2. Подробно о режимах работы:

	savemenu - обходит меню 1С:Розницы, открывая и считывая подменю разделов. Выстроенную иерархию программа сериализует и сохраняет в свою папку с именем dict_main_elements.pickle.

	open_forms - читает файл dict_main_elements.pickle, затем обходит каждое подменю открывая и закрываю форму. После каждого нажатия пытается закрыть все открытое в 1С:Розница и затем снова открыть весь путь до следующего элемента в очереди. Делает скриншот если появилось окно 1С:Предприятие или сообщение пользователю. Пытается их закрыть и продолжить свой обход. При тестировании находит ошибки если, что то с ролями для разных стандартных профилей. Т.е. выгодно запускать под разными пользователями у которых разные профили 1С.  Т.е из папки с программой нужно удалить использованный ранее файл dict_main_elements.pickle, затем в opt.txt ввести режим обхода и пользователя.  Сначало нужно прогнать  savemenu, затем open_forms.

	cursor - бесконечно выводит в консоль координаты браузера с открытой в нем 1С и координаты курсора мыши.

	scenario - примитивный способ создавать универсальные сценарии автотестирования. В каталоге с программой создаем текстовой файл с набором предопределенных команд и записанных координат курсора с помощью режима cursor. Пример создания примитивного сценария и описание доступных команд находится в файле example_scenario.txt.

После того как программа откроет 1С в браузере, нужно ввести в консольное окно имя файла со сценарием (должен быть в каталоге программы). При желании можно легко переписать данный режим для сценарного прощелкивания вообще чего угодно средствами win32api и wscriptshell.

Хоть и убого, но вполне подходит для случаев когда нужно написать, что то быстро, что будет щелкать определенную последовательность в программе. Так же подходит(но нужно дорабатывать) если по каким то причинам не работают стандартные средства сценарного автотестирования.

	go(не работает) - режим который больше всего хотелось реализовать изначально(не рабочий, его нужно переделывать). Пытается обходить все окна и прощелкивать каждый элемент. Хранит иерархию прощелкиваемых таблиц в памяти и сериализует ее в папку с программой при завершении работы. Если при нажатии на элемент открылось новое окно переходит на него и прощелкивает там, потом возвращется к предыдущему. Работает очень медленно. При прикращении работы можно начать обход с того же места если в папке с программой есть файл таблиц data_memory.pickle. Делает скриншоты такие же как в режиме open_forms.

	go_partial(не работает)  - то же самое, что и go только при начале работы нужно указать раздел подменю по которому будет обход. Пример: Администрирование, Обслуживание.



3. Подробнее о программе, ошибки в архитектуре.

	Программа использует три зависимости: pywin32 для эмуляций нажатий мышью и тд., Pillow - участвует в создании скриншотов, Selenium - фреймворк для тестирования браузерных приложений. Все версии зависимостей указаны в файле requirements.txt. Установить зависимоссти можно с помощью setup.py. Так же нужно понимать, что программа недоделана и функционал написан для пробы и дальнейшего использования кусков кода, поэтому могут встречаться ошибки. Реально успешно используются только режимы: savemenu, open_forms под пользователями 1С с разными правами.



Ошибки:

Один плохой класс на всю программу. 

Медленно. 

Неправильное использование try/exception. 

Чрезмерно усложненная реализация. 

Отсутствие юнит тестов.



Что бы я переделал:

Наметил бы архитектуру заранее.

Избавился бы от Pillow.

Использовал бы Селен(Selenium) еще меньше, в конце концов у него есть метод "передать браузеру на исполнение код javascript". Можно было бы таким образом сразу получать нужные данные и уже после этого их сортировать в несколько одновременных процессах. В текущей реализации это не возможно так как несколько параллельных процессов не могут одновременно обращаться к одному и тому же ресурсу с использованием Селена. Это одна из главных проблем медленной работы режима "go".  Если руки дойдут переделаю режим "go" на подобие "open_forms" это не ускорит работу но упростит реализацию и сделает режим хотя бы рабочим.

Реализовать запуск режима "scenario" на мониторе любого размера. Увеличить количество команд. Сделать так что бы сценарии были в отдельной папке и из одного сценария можно было вызывать другой сценарий. Сделать, что бы сценарий работал рекурсивно или повторялся. Сделать так что бы сценарий мог принимать параметры.

Заменить сериализацию таблиц обхода на маленькую базу данных.

Искал более простые и универсальные способы отлавливать нужные окна.



Так же если бы речь шла о простом сайте, а не об 1С приложении - всего этого бы не понадобилось вообще. 

Достатчно было бы одного Селена. 

Так же и у 1С есть более удобные способы автотестирования своих приложений.

      

4. Для работы программы нужны:

Браузер chrome. 

Опубликованная на веб-сервере база 1С:Розницы.

chromedriver.exe в папке с программой.

указать настройки в файле opt.txt в папке с программой.