# Библиотека IIOT  для Codesys 3

Данная библиотека содержит следующие возможносмти:
- JSON [сереализатор](#serealisation) и [дисереализатор](#deserealisation)
- HTTP клиент с возможностью отправки [GET](#21-get-запрос) и [POST](#22-post-запрос) запросов
- [MQTT](#3-mqtt-клиент) клиент работающий по спецификации 3.1.1

<br/>

**`Данная библиотека поддерживается всеми плк програмируемыми в среде Codesys 3  или IDE разработанных на его основе`**<br/>

Ссылки:<br/>
[Библиотека для Codesys V3](./library_codesys3/)<br/>
[Пример использования в CodesysV3 для ОВЕН ПЛК210](./example_codesys3/)

-----
## 1. Сереализация объекта
------
## 1.1 Создание объекта для сереализаци
---

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
|Value    |Values |Структура  хронящая значение **`(Структура преведена ниже)`**   |
|vType   |isTypes  |Перечисление определяющая тип свойства **`(Структура преведена ниже)`**|

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
----
##  1.2 Сереализация объектов в формате JSON
#serealisation
----

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
----
##  1.3 Десереализация объектов в формате JSON
#deserealisation
----
<br/>

Десереализация происходит с помощью функциионального блока `IIOT.FB_JSON_PARSE_ARRAY_ASYNC(xRun:BOOL,JSON:POINTER TO BYTE,OBJECT:POINTER TO BYTE)` где:

|Сойство | Тип | Назначение |
|:--------|-----:|:----------:|
|xRun    |BOOL  | Запуск десереализации **`(Срабатывает по переднему фронту)`**   |
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
__________
## 2. HTTP клиент

_______________

## 2.1 Get запрос
_______________

Get  запрос осуществляется с помощбю функционалного блока `IIOT.HTTP_GET`, данный блок имеет следующие параметры:

|Тип параметра | Название | Тип | Назначение |
|:-----:|:--------|-----:|:----------:|
|Input|sServerName    | String | Hostname или IP адрес  HTTP сервера |
|Input   |wServerPort | Word | Номер порта |
|Input   |sUserName | String(30) | Имя пользователя<br/> *(Применяется при базовой аутоидентификаци)* |
|Input   |sPassword | String(30) | Пароль <br/> *(Применяется при базовой аутоидентификаци)* |
|Input   |abUrlData | Pointer to array of byte | Указатель на строку описывающую URL запроса |
|Input   |uiUrlLength | Uint | Длина HTTP запроса |
|Input   |sUserAgent | String | Имя  HTTP клиента |
|Input   |tTimeOut | Time | Время ожидания ответа |
|In Out  | xSend | Bool | Триггер отправки сообщения <br/> *(Работает по переднему фронту)* |
|Output  | diError| Dint | Ошибка выполнения <br/> *(При успешном выполнении равен нулю)* |
|Output  | uiResponseCode| Uint | Номер ответа <br/> *(При успешном выполнении равен 200 )* |
|Output  | abContentData| Array of byte | Данные ответного сообщения |
|Output  | uiContentLength| Uint | Длина ответного сообщения  |
<br/>

### Объявление переменных:
```
PROGRAM HTTP_Get_Request
VAR
HTTP:IIOT.HTTP_GET;
IP:STRING:='10.177.3.61';
iPort:INT:=80;
url:STRING(255);
run:BOOL;
JSON: STRING(512);
buff:ARRAY[0..IIoT.HTTPCLIENTVARS.MAX_RECEIVE_TCP_CLIENT] OF BYTE;
GetTape:BOOL;
GetTapeStep:INT;
Down:F_Trig;
PARSER:IIoT.FB_JSON_PARSE_ARRAY_ASYNC;
tapes: ARRAY [1..10] OF tapeSettings;
I:INT;
END_VAR
```

###  Программная реализайия:
```
CASE GetTapeStep OF
0:
IF GVL.Http_GetTape THEN
url:=CONCAT('/api/Reels/GetReelForInstrumentByPartName/',DINT_TO_STRING(GVLCoil_01));
IIOT.ClearData(ADR(JSON),510);
GVL.Http_GetTape:=FALSE;
GetTapeStep:=GetTapeStep+1;
END_IF

1:
run:=TRUE;
GetTapeStep:=GetTapeStep+1;

2:
HTTP(
sServerName:= sIp, 
wServerPort:=iPort , 
sUserName:='', 
sPassword:='' , 
sUserAgent:='OWEN PLC210-2',
pabUrlData:=ADR(url) , 
uiUrlLength:=LEN(url) , 
tTimeOut:=T#10S , 
xSend:=run , 
abContentData=>buff , 
uiContentLength=> );
Down(CLK:=RUN);
IF Down.Q  AND (HTTP.uiResponseCode=200) THEN
JSON:=IIOT.IGetString(ADR(buff),512);	
GetTapeStep:=GetTapeStep+1;
END_IF

3:
FOR I:=1 TO 10 DO
tapes[I].Name.Value.toString:='BLANK';
tapes[I].Length.Value.toReal:=0;
tapes[I].Start.Value.toReal:=0;
tapes[I].End.Value.toReal:=0;
END_FOR
PARSER(xRun:=FALSE);
GetTapeStep:=GetTapeStep+1;

4:
PARSER(xRun:=TRUE,JSON:=ADR(JSON),Object:=ADR(settingsVars.Tapes));
IF PARSER.xComplet THEN
PARSER(xRun:=FALSE);
GetTapeStep:=0;
END_IF
END_CASE
```
___________
## 2.2 Post запрос
___________

Post  запрос осуществляется с помощбю функционалного блока `IIOT.HTTP_POST`, данный блок имеет следующие параметры:

|Тип параметра | Название | Тип | Назначение |
|:-----|:--------|-----:|:----------:|
|Input|sServerName    | String | Hostname или IP адрес  HTTP сервера |
|Input   |wServerPort | Word | Номер порта |
|Input   |sUserName | String(30) | Имя пользователя<br/> *(Применяется при базовой аутоидентификаци)* |
|Input   |sPassword | String(30) | Пароль <br/> *(Применяется при базовой аутоидентификаци)* |
|Input   |sUrl | String(200) | адрес URL запроса |
|Input   |pabReqData | Pointer to array of byte | Указатель на данные HTTP запроса |
|Input   |uiReqDataCount | Uint| Длина сообщения |
|Input   |sUserAgent | String | Имя  HTTP клиента |
|Input   |sContentType | String | тип содержимого <br/> *(По умолчанию `application/json` )*|
|Input   |sUserAgent | String | Имя  HTTP клиента |
|Input   |tTimeOut | Time | Время ожидания ответа |
|In Out  | xSend | Bool | Триггер отправки сообщения <br/> *(Работает по переднему фронту)* |
|Output  | diError| Dint | Ошибка выполнения <br/> *(При успешном выполнении равен нулю)* |
|Output  | uiResponseCode| Uint | Номер ответа <br/> *(При успешном выполнении равен 200 )* |
|Output  | abContentData| Array of byte | Данные ответного сообщения |
|Output  | uiContentLength| Uint | Длина ответного сообщения  |
<br/>

### Объявление переменных:
```
PROGRAM HTTP_Post_Request
VAR
HTTP:IIoT.HTTP_POST;
xSent:BOOL;
sIP:STRING:='10.177.3.61';
iPort:INT:=80;
url:STRING(255);
body:STRING:='{}';
NewProcess:BOOL;
EndProcess:BOOL;
process_id:INT:=210;
run_number:INT:=300;
END_VAR

```
###  Программная реализайия:
```

IF GVL.Http_NewProcess THEN
url:=concat('/api/Opc/ProcessStart/',DINT_TO_STRING(gvl.Process_ID));
url:=concat(url,'/run/');
url:=concat(url,INT_TO_STRING(GVL.Run_Number));
url:=concat(url,'/sendReel/');
url:=concat(url,DINT_TO_STRING(gvl.Coil_01));
url:=concat(url,'/rcvReel/');
url:=concat(url,DINT_TO_STRING(gvl.coil_02));
GVL.Http_NewProcess:=FALSE;
xSent:=TRUE;
END_IF

IF  GVL.Http_EndProcess THEN
url:=concat('/api/Opc/ProcessStop/',DINT_TO_STRING(gvl.Process_ID));
url:=concat(url,'/run/');
url:=concat(url,INT_TO_STRING(GVL.Run_Number));
url:=concat(url,'/sendReel/');
url:=concat(url,DINT_TO_STRING(gvl.Coil_01));
url:=concat(url,'/rcvReel/');
url:=concat(url,DINT_TO_STRING(gvl.Coil_02));
GVL.Http_EndProcess :=FALSE;
xSent:=TRUE;
END_IF

HTTP(
sServerName:=sIp , 
wServerPort:= iPort, 
sUserName:='*' , 
sPassword:='*', 
sUrl:= url, 
pabReqData:=ADR(body) ,
uiReqDataCount:=len(body) , 
sContentType:='application/json',
sUserAgent:='OWEN PLC210-2',
tTimeOut:=T#10S , 
xSend:=xSent);
```
------
## 3. MQTT клиент
----
## 3.1 Описание блока
------
Работа с Mqtt брокером реализуется с помощью блока `IIOT.FB_MQTTClient`.
<br/>

Данный блок имеет следующие входы и выходы:
|Тип параметра | Название | Тип | Назначение |
|:-----|:--------|-----:|:----------:|
|Input|xEnable| Bool |  Активация соеденения|
|Input|sBrokerAddress| String | Адрес брокера  <br/>*(hostname или IP)* |
|Input|uiPort| Uint | Номер порта <br/>*(1883 по-умолчанию)* |
|Input|sUsername| String |  Имя пользователя |
|Input|sPassword| String |  Пароль|
|Input|sWillTopic| String | Топик последнего аварийного сообщения |
|Input|sWillMessage| String |  Последнее аварийное сообщение|
|Input|sWillRetain| String | Флаг сохранения последнего сообщения |
|Input|xAutoReconnect| Bool | Переподключение клиента |
|Input|sPayload| String(1024) |  Полезная нагрузка  сообщения|
|Input|sTopicPublish| String |  Топик публикации|
|Input|sTopicSubscribe| String | Топик подписки |
|Input|xRetain| Bool | Сохранять опубликованное сообщение |
|Input|xPublish| Bool | Сигнал публикации сообщения в топик `sTopicPublish`  <br/> *(Работает по переднему фронту)*  |
|Input|xSubscribe| Bool | Сигнал подписки на сообщения топика `sTopicSubscribe`<br/> *(Работает по переднему фронту)*  |
|Output  | sLastReceivedMessage| String(1024) | Последнее полученное сообщение из подписанных топиков |
|Output  | sLastReceivedMessageTopic| String(1024) | Топик последнего полученного сообщения |
|Output  | arsLastReceivedMessages| Array [0..24] of String(1024) | Список последних 25 полученных сообщений|
|Output  | arsLastReceivedMessageTopics| Array [0..24] of String(1024) | Список последних 25 топиков|
|Output  | xReceivedMessageNotification| Bool | Уведомление о полученном сообщении|
|Output  | sDiagMsg| String | Сообщение диагностики|
|Output  | udiState| Udint | Слово состояние блока|
|Output  | xError| bool | Ошибка в процессе выполнения|

<br/>

-----

## 3.2 Настройка подключения MQTT клиента

------

## 3.2.1 Создание структур

-----

Для реализации  взаимодействия с блоком `IIOT.FB_MQTTClient` необходимо создать  структуру `Client_Mqtt` :
```
TYPE Client_Mqtt :
STRUCT
	xEnableMqtt:BOOL;
	Payload:STRING(1024);
	TopicPublish:STRING;
	TopicSubscribe:STRING;
	Publish:BOOL;
	Subscribe:BOOL;
	LastReceivedMessage:STRING(1024);
	LastReceivedTopic:STRING(1024);
	ReceivedMessageNotification:BOOL;
END_STRUCT
END_TYPE
```
Для взаимодействия с обработчиком полученных сообщений необходимо создать структуру  `MessageQ`  :
```
TYPE MessageQ :
STRUCT
	Topics:POINTER TO STRING;
	Payload:POINTER TO BYTE;
END_STRUCT
END_TYPE
```
Для хранение настроек подключения необходимо создать  структуру `MqttUser` :
```
TYPE MqttUser :
STRUCT
Login:STRING;
Password:STRING;
Port:INT;
Host:STRING;
END_STRUCT
END_TYPE
```
Добавление переменных и необходимых структур выполняется в следующем виде:
```
VAR_GLOBAL
inTopics:ARRAY[1..50] OF STRING;
INIT:BOOL;
inMsg:MessageQ;
ChangeSettings:BOOL;
LoginData:MqttUser:=(Login:='mqtt',Password:='Superox495',port:=1883,host:='194.87.46.139');
Mqtt:Client_Mqtt;
END_VAR
```
-----

## 3.2.2 Инциализация MQTT клиента

----

Для создания необходимых подписок необходимо собрать программу 
инциализации  `INIT`:<br/>
### Описание переменых:

```
PROGRAM INIT
VAR
delay:TON;
delay_2:TON;
xSubscribe:BOOL;
i:int;
END_VAR
```
### Реализация программы:
```
IF mqttVars.INIT THEN
MqttVars.inTopics[1]:='/id1/Color/Settings/Hvac/1';
MqttVars.inTopics[2]:='/id1/Color/Settings/Hvac/2';
MqttVars.inTopics[3]:='/id1/Color/Settings/Hvac/3';
MqttVars.inTopics[4]:='/id1/Color/Settings/Hvac/4';
MqttVars.inTopics[5]:='/id1/Color/Settings/Hvac/5';
MqttVars.inTopics[6]:='/id1/Color/Settings/Hvac/6';
MqttVars.inTopics[7]:='/id1/Color/Settings/Hvac/7';

mqttVars.INIT:=FALSE;
mqttVars.mqtt.xEnableMqtt:=TRUE;
i:=0;
END_IF

IF NOT mqttVars.INIT THEN
delay(in:=xSubscribe ,pt:=T#1000MS);
delay_2(in:=delay.Q,pt:=T#1000MS);

IF NOT xSubscribe AND NOT delay_2.Q AND mqttVars.mqtt.xEnableMqtt AND i<1 THEN
i:=i+1;
mqttVars.mqtt.TopicSubscribe:='/id1/Color/Settings/#';

xSubscribe:=TRUE;
END_IF

IF delay_2.Q THEN
xSubscribe:=FALSE;
END_IF

mqttVars.mqtt.Subscribe:=delay.Q;
END_IF;
```

----

## 3.2.3 Создание коннектора

-----

Для реализации функции подключения к брокеру сообщения необходимо создать программу `MQTT_Connector`:
### Описание переменных:

```
PROGRAM MQTT_Connector 
VAR
fbMqttClient: IIOT.FB_MqttClient;
sLastReceivedMessage:STRING(1024);
sLastReceivedTopic:STRING(1024);
i:INT;
delay:ton;
END_VAR
```
### Реализация программы:
<br/>


```
fbMqttClient(
	xEnable:= mqttVars.Mqtt.xEnableMqtt, 
	sBrokerAddress:= mqttVars.LoginData.Host, 
	uiPort:=mqttVars.LoginData.Port , 
	sUsername:=mqttVars.LoginData.Login , 
	sPassword:=mqttVars.LoginData.Password , 
	sWillTopic:= , 
	sWillMessage:= , 
	sWillRetain:= , 
	xAutoReconnect:=TRUE , 
	sPayload:=mqttVars.Mqtt.Payload, 
	sTopicPublish:=mqttVars.Mqtt.TopicPublish, 
	sTopicSubscribe:=mqttVars.Mqtt.TopicSubscribe , 
	xRetain:= , 
	xPublish:=mqttVars.Mqtt.Publish , 
	xSubscribe:=mqttVars.Mqtt.Subscribe , 
	sLastReceivedMessage=>mqttVars.Mqtt.LastReceivedMessage , 
	arsLastReceivedMessages=> , 
	arsLastReceivedMessageTopics=> , 
	xReceivedMessageNotification=> mqttVars.Mqtt.ReceivedMessageNotification, 
	sDiagMsg=> , 
	udiState=> , 
	xError=>, );

delay(in:=mqttVars.Mqtt.Publish,PT:=T#50MS);
IF delay.q THEN
mqttVars.Mqtt.Publish:=FALSE; 
END_IF
	
```
----
## 2.2.4 Обработка сообщений поступивших с пабликов
----
<br/>

Для обработки сообщений нужно создать программу обработки поступающих сообщений  `GetHeandler_Mqtt`:

### Описание переменных:
```
PROGRAM GetHeandler_Mqtt 
VAR
I:INT;
Msg:MessageQ;
hvac:hvacSettings;
END_VAR
```
### Реализация программы:
```
IF mqttVars.Mqtt.ReceivedMessageNotification  THEN
Msg.Topics:=ADR(mqttVars.Mqtt.LastReceivedTopic);
Msg.Payload:=ADR(mqttVars.Mqtt.LastReceivedMessage);
MqttVars.ChangeSettings:=TRUE;
END_IF


IF MqttVars.ChangeSettings THEN
FOR I:=1 TO 4 DO	
IF MqttVars.inTopics[i]=Msg.Topics^ THEN
EXIT;
END_IF
END_FOR
CASE i OF

1:
SettingsVars.HvacSettings[1].TagName.Value.toString:='test';
IIOT.JSON_TO_Object(ADR(SettingsVars.HvacSettings[1]),Msg.Payload);
MqttVars.ChangeSettings:=FALSE;	

2:
SettingsVars.HvacSettings[2].TagName.Value.toString:='test';
IIOT.JSON_TO_Object(ADR(SettingsVars.HvacSettings[2]),Msg.Payload);
MqttVars.ChangeSettings:=FALSE;	

3:
SettingsVars.HvacSettings[3].TagName.Value.toString:='test';
IIOT.JSON_TO_Object(ADR(SettingsVars.HvacSettings[3]),Msg.Payload);
MqttVars.ChangeSettings:=FALSE;	
4:
SettingsVars.HvacSettings[4].TagName.Value.toString:='test';
IIOT.JSON_TO_OBJECT(ADR(SettingsVars.HvacSettings[4]),Msg.Payload);
MqttVars.ChangeSettings:=FALSE;	
END_CASE
END_IF
```

----
## 2.2.5  Публикация  текущих показаний в топики
----
<br/>

Для публикации текущих показаний в топики mqtt  брокера  нужно  организовать программу  выполняющую публикации `Publication_Mqtt`:

### Описание переменных:
```
PROGRAM Publication_Mqtt 
VAR
	Step:INT;
	delay:ton;
	down:F_TRIG;
	i:INT;
END_VAR
```
<br/>

### Реализация программы:
```
IF mqttVars.Mqtt.xEnableMqtt THEN
CASE Step OF
0:
Step:=Step+1;

1:
IF NOT mqttVars.Mqtt.Publish THEN	
FOR i:=1 TO 7 DO
mqttVars.hvacData[i].TagName.Value.toString:=Concat('Hvac',INT_TO_STRING(i));
mqttVars.hvacData[i].Timestamp.Value.toDint:=DT_TO_DINT(TargetVars.stRtc.dtDateAndTime);	
END_FOR
mqttVars.Mqtt.TopicPublish:='/id1/Color/Monitor/Hvac/';
mqttVars.Mqtt.Payload:=IIOT.ARRAY_TO_JSON(ADR(mqttVars.hvacData)); // Серелезация данных
mqttVars.Mqtt.Publish:=TRUE;
Step:=Step+1;
END_IF

2:
down(CLK:=mqttVars.Mqtt.Publish);
IF down.Q THEN
Step:=Step+1;
END_IF
ELSE
IF NOT mqttVars.Mqtt.Publish THEN
delay(IN:=TRUE,PT:=T#1S);
IF delay.Q THEN
delay(IN:=FALSE);
Step:=0;	
END_IF
END_IF

END_CASE
END_IF
```

<br/>
