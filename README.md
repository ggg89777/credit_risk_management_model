﻿<a name="br1"></a> 

**Модель кредитного**

**риск-менеджмента**

Алексей Козлов

2023 год



<a name="br2"></a> 

**Описание проблем**

***обучения модели кредитного скоринга по данным из транзакций клиентов***

• Датасет закодирован и практически невозможен анализ

(все данные фактически являются **категориальными**)

• Из-за большого размера датасета (26162717 строк)

необходима **оптимизация** по использованию оперативной

памяти

• Данные в датасете **не сбалансированы** (2893558 – класс 0,

106442 – класс 1)

• Стандартными классами из пакета sklearn невозможно

создать pipeline c кодированием категориальных

переменных **ohe + агрегация данных**

• Необходимо получить **ROC-AUC > 0,75** на тестовой

выборке



<a name="br3"></a> 

Feature Preparation – 0

Признаки, над которыми проводились преобразования

• **rn** - порядковый номер кредитного

продукта

• **pre\_loans\_total\_overdue** - текущая

просроченная задолженность

• **is\_zero\_util – флаг**: отношение оставшейся

невыплаченной суммы кредита к

кредитному лимиту равно 0

• **pre\_loans\_credit\_limit** - кредитный лимит



<a name="br4"></a> 

Feature Preparation – 1

порядковый номер кредитного продукта

rn

**.groupby(“id”)**

**+ agg(“count”)**

Признак – количество кредитов в

кредитной истории



<a name="br5"></a> 

Feature Preparation – 2

текущая просроченная задолженность

**pre\_loans\_total\_overdue**

**Операция**

**удаления**

**признака**



<a name="br6"></a> 

Feature Preparation – 3

отношение оставшейся невыплаченной суммы кредита к

кредитному лимиту равно 0

**is\_zero\_util**

**Инвертирование**

**.groupby(“id”)**

**+ agg(“sum”)**

Признак – количество действующих

кредитов



<a name="br7"></a> 

Feature Preparation – 4

кредитный лимит

**pre\_loans\_credit\_limit**

**Удаление дубликатов**

**Удаление дубликатов**

**(оставить только**

**первый кредит)**

**(оставить только**

**последний кредит)**

**Операция получения**

**разницы между**

**первым и последним**

**кредитом**

Признак – изменение кредитного лимита



<a name="br8"></a> 

Feature Preparation – 5

Все остальные признаки

**Категориальные признаки**

**OneHotEncoder**

**.groupby(“id”) +**

**agg(“sum”)**

**Сжатый по id датасет, в котором сохраняется информация по**

**кредитам**



<a name="br9"></a> 

Презентация финального датасета

какие из признаков были добавлены в модель

**Только ohe + agg**

ROC\_AUC = 0.7627924466510313

ROC\_AUC = 0.7627924466510313

Удаление дает экономию памяти

ROC\_AUC = 0.7627924466510313

**rn**

**pre\_loans\_total\_overdue**

**open\_loans**

**growth\_limit**

ROC\_AUC = 0.7639072689224383

• В итоге только признак **growth\_limit** улучшает

модель и попадает в итоговый датасет



<a name="br10"></a> 

**Презентация результатов моделирования - 0**

**результаты применения различных моделей (на части df)**

q **LGBMClassifier +**

**class\_weight**

q **CatBoostClassifier +**

**Upsampling**

ROC train: 0. 829 …

ROC test: **0. 7639** ….

Precision test: 0. 069 …

Recall test: 0. 6994 …

[[34014 14439]

q **MLPClassifier +**

**SMOTE**

ROC train: 0. 890…

ROC test: **0. 756**….

Precision test: 0. 076…

Recall test: 0. 5985…

[[37200 11253]

q **XGBClassifier +**

**Upsampling**

ROC train: 0.805…

ROC test: **0.739**….

Precision test: 0.137…

Recall test: 0.0025…

[[ 48428 25]

[

465 1082]]

ROC train: 0. 932…

ROC test: **0. 716**….

Precision test: 0. 075…

Recall test: 0. 4951…

[[39100 9353]

[

621 926]]

[

1543 4]]

[

781 766]]

q В результате у **LGBMClassifier** с использованием **увеличенных штрафов** за класс 1 получилась

наилучшая метрика **ROC\_AUC = 0,7639** и эта модель в дальнейшем использовалась в проекте.

q В результате **подбора гиперпараметров** получилась лучшая модель со следующими параметрами:

**class\_weight={0:1, 1:32}, reg\_lambda = 1000, reg\_alpha = 0.4**



<a name="br11"></a> 

**Презентация результатов моделирования - 1**

**агрегирование датасета как гиперпараметр модели**

• Логично предположить, что **не все кредиты** в кредитной

истории несут в себе полезную информацию для определения

финансовой состоятельности клиента.

• Операцией **.groupby(['id']).tail(n)** можно обрезать датасет до

• n – последних кредитов в кредитной истории.

• В результате, обрезая df до последних n-кредитов, меняются

метрики модели, тем самым, этот параметр является

**гиперпараметром модели**.

• Результаты экспериментов отсортированные по **ROC\_AUC:**

ü **5 последних кредитов - 0.7649534783312695**

ü **8 последних кредитов - 0.7648794957084007**

ü **23 последних кредитов - 0.7644541239765721**

ü **. . .**

ü **. . .**



<a name="br12"></a> 

**Презентация результатов моделирования - 2**

**ансамбль из предсказаний моделей**

**df c последними 5 кредитами**

**df c последними 8 кредитами**

**df c последними 23 кредитами**

**Class Pipes**

**.fit**

**.fit**

**.predict**

**.predict**

**predict\_proba – \*0.83, 0.42, ….+ predict\_proba – \*0.79, 0.35, ….+**

**predict\_proba – \*0.85, 0.40, ….+**

**redict\_proba.mean() – \*0.82, 0.39, ….+**

**ROC\_AUC на одном файле pq = 0.7696446676859472**



<a name="br13"></a> 

**Презентация результатов моделирования - 3**

**оптимизация**

•

•

•

•

В итоговом датафрейме присутствует **26 миллионов строк**.

Если использовать их в стоковом виде, то возникнут проблемы с использованием **оперативной памяти**.

При загрузке одного pq-файла требуется около **900 мб. оперативной памяти**.

Если изменить тип данных всех признаков кроме id на **int8** (так как все данные являются категориальными)

то размер датасета сокращается до **100 мб**. Но при загрузке **12 файлов** с изменением типа данных

получится около **1,2 гб**.

•

•

Поэтому на этапе склейки датафрейма применяется функция **convert\_parquet,** которая загружает файл,

меняет ему тип данных на **int8** и конкатенирует его с предыдущим оптимизированным файлом. Финальный

датафрейм сохраняется на диск (так как в pq **сохраняется тип данных**, при загрузке с диска никаких больше

преобразований не требуется).

При дальнейшей обработке файла опять возникает проблема оптимизации из-за увеличения размерности

датафрейма после **OneHotEncoder (необходимо более 85 гб оперативной памяти),** так как по умолчанию

ohe **конвертирует данные в float64.** Поэтому при обработке ohe применяется параметр dtype = np.int8. В

результате для преобразования ohe необходимо примерно **10 гб оперативной памяти.**

•

•

Обработать датафрейм **OneHotEncoder по 1 признаку** для экономии памяти **не представляется возможным**

так как для встраивания в **PipeLine** необходим список фичей для корректной работы.

При использовании метода .**groupby** pandas так же преобразует тип данных. Но при его использовании в

**PipeLine** каждую колонку можно обработать итеративно и изменить тип данных каждой колонки c помощью

.astype(np.int8), так как параметры этого преобразования не привязаны к объекту **PipeLine.**

• **ВЫВОД : В результате оптимизации в проекте используется 3 сконкатенированных и**

**оптимизированных датасета (потребление не более 5 гб оперативной памяти). На этапе обработки при**

**кодировании используется dtype = np.int8 (не более 10 гб на одном итеративно обрабатываемом**

**датасете), при агрегировании итеративная обработка по 1 столбцу с изменением типа данных на int8**

**(потребление памяти незначительно).**

• **Пиковое потребление оперативной памяти не превышает 30 гб с учетом**

**7 гб, которая занимает ОС WINDOWS 10**



<a name="br14"></a> 

**Демонстрация конечного результата**

**итоговые метрики**

**ROC\_AUC на test 0 файла pq ROC\_AUC на X\_test всего df ROC\_AUC на X\_train + X\_test**

**0.7696446676859472**

**0.7611946467243675**

**0.7674799730057711**

