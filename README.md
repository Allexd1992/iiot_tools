# Библиотека IIOT  для Codesys 3
Данная библиотека поддерживается всеми плк програмируемыми в среде Codesys 3  или IDE разработанных на его основе<br/>

[Библиотека для Codesys V3](./library_codesys3/IIOTv2.compiled-library)
## Создание объекта для сереализации:

Объекты создаются на бале типа `IIOT.Tag` который имеет следующую структуру:
```
TYPE Tag :
STRUCT
	Name:STRING(20);
	Value:Values;
	{attribute 'hide'}vType:isTypes;
END_STRUCT
END_TYPE
```
|Сойство | Тип | Назначение |
|:--------|-----:|:----------:|
|Name    |String  |Название свойства **`(Должно совпадать с названием свойства)`**     |
|Value    |Values |Структура  хронящая значение **`(Структура премедена ниже)`**   |
|vType   |isTypes  |Перечисление определяющая тип свойства **`(Структура премедена ниже)`**|

Тип Values имеет следующий вид:
```
TYPE Values :
UNION
	toString:STRING(20);
	toInt:INT;
	toBool:BOOL;
	toReal:REAL;
	toDint:DINT;
END_UNION
END_TYPE
```
Перечисление isTypes имеет следующий вид:
```
TYPE isTypes :
(
	isNull:=-1,
	isREAL := 0,
	isString:=1,
	isInt:=2,
	isDint:=3,
	isBool:=4
);
END_TYPE
```
Создание объекта реализуется с помощью  объявления новой структуры:

```
TYPE tapeSettings : 
STRUCT
	Name:IIOT.Tag:=(Name:='Name',vType:=IIOT.isTypes.isString);
	Length:IIOT.Tag:=(Name:='Length',vType:=IIOT.isTypes.isReal);
	Start:IIOT.Tag:=(Name:='Start',vType:=IIOT.isTypes.isReal);
	End:IIOT.Tag:=(Name:='End',vType:=IIOT.isTypes.isReal);
	Numb:IIOT.Tag:=(Name:='Numb',vType:=IIOT.isTypes.isInt);
	Id:IIOT.Tag:=(Name:='Id',vType:=IIOT.isTypes.isDint);
	{attribute 'hide'}Reserve1:IIOT.Tag:=(Name:='Reserve',vType:=IIOT.isTypes.isString);
	{attribute 'hide'}Reserve2:IIOT.Tag:=(Name:='Reserve',vType:=IIOT.isTypes.isString);
	{attribute 'hide'}Reserve3:IIOT.Tag:=(Name:='Reserve',vType:=IIOT.isTypes.isString);
	{attribute 'hide'}Reserve4:IIOT.Tag:=(Name:='Reserve',vType:=IIOT.isTypes.isString);
	{attribute 'hide'}Reserve5:IIOT.Tag:=(Name:='Reserve',vType:=IIOT.isTypes.isString);
	{attribute 'hide'}Reserve6:IIOT.Tag:=(Name:='Reserve',vType:=IIOT.isTypes.isString);
	{attribute 'hide'}Reserve7:IIOT.Tag:=(Name:='Reserve',vType:=IIOT.isTypes.isString);
	{attribute 'hide'}Reserve8:IIOT.Tag:=(Name:='Reserve',vType:=IIOT.isTypes.isString);
	{attribute 'hide'}Reserve9:IIOT.Tag:=(Name:='',vType:=-1);
END_STRUCT
END_TYPE
```
***
``Важно!`` В объявляемом объекте должно быть 15 свойств ``IIOT.Tag`` если необходимых свойств меньше то необходимо
добавить свойства в следующем виде:<br/> `{attribute 'hide'}Reserve1:IIOT.Tag:=(Name:='Reserve',vType:=IIOT.isTypes.isString);`<br/> Hазвание свойства может быть любым.
***
Объявление объекта в проекте реализуется на основе созданной структуры: 
```
VAR_GLOBAL
	tapes:ARRAY[1..20] OF tapeSettings;
	null:IIOT.nullObject;
END_VAR
```

***
``Важно!`` После каждого объекта нужно добавлять  null-объект `IIOT.nullObject` для корректной работы сереализатора и дисереализатора
***
##  Сереализация и десереализация объектов в формате JSON


Сереализация происходит с помощью функции `IIOT.ARRAY_TO_JSON(pointer object)`<br/>
<br/>
Переменные программы:
```
PROGRAM Serealisation_Json
VAR
    STEP:INT;
    JSON:STRING(2048);
    POINT:POINTER TO BYTE;
    SEREALISATION:BOOL;
END_VAR
```
Реализация программы:
```
CASE STEP OF
0:
IF SEREALISATION THEN
SEREALISATION:=FALSE;
STEP:=STEP+1;
END_IF

1:
IIOT.ClearData(ADR(JSON),2048);
JSON:=IIOT.array_TO_JSON(ADR(settingsVars.tapes));
STEP:=STEP+1;

2:
FB(xExecute:=FALSE);
IF NOT FB.xComplet THEN
STEP:=0;	
END_IF
```

Десереализация происходит с помощью функциионального блока `IIOT.FB_JSON_PARSE_ARRAY_ASYNC(xRun:BOOL,JSON:POINTER TO BYTE,OBJECT:POINTER TO BYTE)` где:

|Сойство | Тип | Назначение |
|:--------|-----:|:----------:|
|xRun    |BOOL  | Запуск десереализации **`(Срабатывает по переднему фронту **`(Структура премедена ниже)`**   |
|JSON   |POINTER TO BYTE  |Cсылка на JSON строку |
|Object   |POINTER TO BYTE  | Ссылка  на сереализуемый объект |
|xComplet   |BOOL  | Сигнал завершения процесса дисереализации |
|xError   |BOOL  | Статус  ошибки выполнения дисереализации |


Переменные программы:


```
PROGRAM JSON_PARSER
VAR

PARSER:IIOT.FB_JSON_PARSE_ARRAY_ASYNC;
FB:FB_FILE_READ;
btn5:BOOL;
STEP:INT;
pJson: POINTER TO STRING(10000);
JSON:STRING(10000);
clearJson:STRING(10000);
i:INT;
END_VAR
```
Реализация программы:

```
CASE Step OF
0: 
IF SettingsVars.READ_TAPES_SETTINGS_SW THEN
STEP:=STEP+1;
END_IF
1:

PARSER(xRun:=FALSE);
IF NOT Parser.xComplet THEN
STEP:=STEP+1;
END_IF
	
2:
PARSER(xRun:=TRUE,JSON:=ADR(clearJson),OBJECT:=ADR(settingsVars.tapes));
IF PARSER.xComplet THEN
PARSER(xRun:=FALSE);
STEP:=0;
SettingsVars.READ_TAPES_SETTINGS_SW:=FALSE;
END_IF
	
END_CASE
```



