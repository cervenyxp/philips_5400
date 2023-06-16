# ESP-Philips-5400-Coffee-Machine

Модификация и удаленное управление кофемашиной Philips 5400. Управление реализовано средствами ESPHome. Проект будет дорабатываться и дополняться новыми сведениями.

> Важно!
Все материалы этого проекта (прошивки, схемы, 3D модели и т.п.) предоставляются "КАК ЕСТЬ". Всё, что вы делаете с вашим оборудованием, вы делаете на свой страх и риск. Автор не несет ответственности за результат и ничего не гарантирует. Модификация кофемашины требует вмешательства, что лишит автоматически вас гарантии на новую кофемашину.

Вы можете принять участие в развитие проекта, например написать код для имитации счетчика чтобы можно было готовить кофе много раз, а не один раз после включения. Об этом будет написано ниже.

Что на текущий момент реализовано?

1) Разобрал протокол кофейных напитков и сенсоров
2) Реализовал удаленное включение и выключение
3) Написал код для вывода различных сенсоров и приготовления кофейного напитка
4) Отвод воды от носика

Что не реализовано? 
1) Написать код, который бы имитровал работу счетчка и можно было отправлять команду приготовление кофейного напитка много раз. Нужна помощь. Я не знаю как это сделать. Я не являюсь программистом и не имею опыта в реверсинжиниринге, это мой первый опыт и проект. Благодаря поддержки сообщества моего чата [Умный дом с Диваном](https://t.me/smart_home_divan) я смог расшифровать и реализовать удаленное управление кофемашиной


<details>
  <summary>Снимаем панель управления и встраиваем ESP</summary>
  

Снимаем панель управления. Снимается не сложно. Чуть тянем на себя, в эти точки вставляем пластиковую лопатку и вынимаем панель

![image](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/37297b8c-7ed0-4eba-a87c-d1d52633bff0)

Нам нужен доступ к 8-ми жильному проводу, который нужно подключить к ESP. 

![image](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/66c80f26-1e77-490c-abf9-e8714394c3a0)
![1682532453964](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/6e8051d0-6949-4ae1-b198-7de81f267a88)
![image](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/453cb14a-18e4-40db-9bb3-2d89452d8a69)


> Подключаем по схеме на выбор, через реле или через оптопару. Сервопривод подключаем по желанию, он нужен для работы отвода воды от носика.

Схема с подключением реле 5В (KY-019)
![image](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/41beae4d-7b4d-4d71-a93b-fa1ecb37ee06)

Схема с подключением оптопары PC817C
![image](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/262283be-e9d7-4294-9922-9b239c1d9ac3)

</details>


<details>
  <summary>Разбираем панель управления</summary>
  

Почему именно я разобрал панель управления и подключил реле, и почему не оптопару? Были попытки найти решение включить кофемашину без разборки панели управления. Пробовал размыкать питание через транзистор, пробовал посылать посылать команды, но это было безуспешно. 

Подключал резистор по такой схеме, но этот способ работает на кофемашине серии 2000/3000, но не подошел для серии 5400. Для справки, есть два проекта для кофемашин серии 2000, это [ESPHome-Philips-Smart-Coffee]( https://github.com/TillFleisch/ESPHome-Philips-Smart-Coffee) и [SmartPhilips2200](https://github.com/chris7topher/SmartPhilips2200)

![image](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/ecd55f9c-edc0-495a-a7f1-23a613b9e759)

Снимаем наклейку с чипа и видим маркировку с названием чипа STM32L4R5VIT6

![1684769136876](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/c3b6328c-4986-4013-83d3-302b8aa27684)
![1684769136753](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/4b79edb0-6a39-43e6-8098-d7f6085c2761)

В документации чипа [STMicroelectronics STM32L4R5VIT6](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/blob/main/Documentation/STMicroelectronics%20STM32L4R5VIT6.pdf) находим наш чип, это STM32L4R5VIT6 LQFP100 (25 ножек с каждой стороны)

![image](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/be6c3d65-1ae6-4b33-8558-b67b3ff681e0)

В документации пишется, что управлять кофемашиной байтами можно после включения, поэтому у меня и не получилось включить ее удаленно. Возможно я ошибаюсь и если будет найдено такое решение, то обязательно внесу поправку

![image](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/e369a60d-bf57-4f5f-884b-d150b73455b2)


***

Для имитации работы сенсорной кнопки можно использовать модуль реле одноканальный 5В (KY-019) или оптопару PC817C, которые будут подавать GND на резистор R110, к которому идет дорожка от кнопки питания, тем самым включая и выключая кофемашину. Я не стал рисковать припаивать провод после резистора R110, так как после резистора идет прамя дорожка к ножке чипа и я не знаю как чип может на это отреагировать, но, если так сделать, то тогда у нас будет возможность включать кофемашину через сенсорную кнопку и удаленно. Это было проверенно на сенсорной лампе, где я припаял провода после резистора и я мог включать и выключать удаленно, а также включать и выключать лампу нажимая на сенсорную кнопку. На кофемашине я не решаюсь это повторить.

Панель управления в разборе
![image](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/dc1a069f-0424-4e9b-85df-5aabd3d19b64)
![image](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/a28be6e2-095a-4267-924d-a6f1bac9e765)

Дорожка от кнопки питания к резистору R110
![image](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/b9eac07a-68c4-4fc9-8116-e4c28f6e2309)
![image](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/48e1a6c9-0756-4260-8f25-3db06706eeea)
![image](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/dc5914ef-7bc0-4473-9e51-794e4e1dab9a)

Припаиваем провод к резистору R110
![image](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/87aae2a6-bc5d-417c-837e-8faa30682a70)

Делаем отверстие в крышке и выносим провод наружу, который подключим к реле или к оптопаре
![image](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/2e36193d-ceb9-4408-856e-d085791d2b71)

  
</details>


<details>
  <summary>Расшифровка протокола</summary>
  
Протокол кофейного напитка
![image](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/a04845ff-e944-417e-9c2e-4f34a012ccc8)
![image](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/c72aaee5-8630-458e-9adf-b1c1255909a1)


Протокол системных сенсоров
![image](https://github.com/DivanX10/ESP-Philips-5400-Coffee-Machine/assets/64090632/c24d8cdc-02ed-44eb-8b4b-5dc1f468412b)

</details>


