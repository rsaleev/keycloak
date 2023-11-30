## Аутентификация authorization_code -\> token

### Шаг 1. Получение authorization_code

пользователь отправляется на форму авторизации в КК по этому адресу. В параметрах передается url куда он вернется с `code`

***GET*** *`/realms/SST/protocol/openid-connect/auth`*

##### *Параметры запроса*

#|
||

Параметр

|

Обязательный

|

Пример

|

Примечание

||
||

client_id

|

да

|

account-console

|

Идентификатор клиента

||
||


|


|

client-backoffice

|


||
||

redirect_uri

|

да

|

https://kk.dev.selftech.ru/realms/SST/account/#/

|

После завершения взаимодействия с владельцем ресурса сервер авторизации направляет пользовательский агент владельца ресурса обратно клиенту. Сервер авторизации перенаправляет пользовательский агент на конечную точку перенаправления клиента, ранее установленную сервером авторизации в процессе регистрации клиента или при отправке запроса на авторизацию. Вы настроите redirect_uri при создании нового клиента OAuth на сервере авторизации Keycloak. Если вы включаете этот параметр запроса в запрос, то его значение должно соответствовать значению, настроенному на сервере авторизации Keycloak. В противном случае произойдет ошибка.

Callback должен получить значение`authorization_code`

||
||


|


|

http://oms.selftech.ru/auth

|


||
||

response_mode

|

нет

|

fragment

|


||
||


|


|


|


||
||

response type

|

да

|

code

|

Это значение должно быть “code”, чтобы поток предоставления кода OAuth работал. Если вы укажете здесь другое значение, запрос не будет выполнен. Response_type является обязательным параметром в потоке предоставления кода OAuth,

||
||


|


|


|


||
||

scope

|

нет

|

openid

|

Область действия запроса на доступ: *openid/profile/…*

||
||


|


|


|


||
||

state

|

нет (рекомендовано)

|

\<UUID, String\>

|

Это рекомендуемое, непрозрачное значение, используемое клиентом для поддержания состояния между запросом и обратным вызовом. Вам нужно будет сгенерировать эту случайную буквенно-цифровую строку символов и включить их в запрос. Сервер авторизации включает это значение при перенаправлении пользовательского агента обратно клиенту. Этот параметр СЛЕДУЕТ использовать для предотвращения подделки межсайтовых запросов. Странице, которая обрабатывает ответ с сервера авторизации, необходимо будет прочитать это значение и сравнить его с исходным, которое было отправлено вместе с этим запросом. Эти два значения должны совпадать.

||
||


|


|


|


||
||

code_challenge

|

нет

|

\<String\>

|

любое сгенерированное значение

||
||


|


|


|


||
||

code_challenge_method

|

нет

|

S256/S512/RS256/RS512

|

алгоритм шифрования `code_challenge`

||
|#

Пример полученного запроса ресурсом, указанным в `redirect_uri`

***GET*** *`/callback?state=fj8o3n7bdy1op5&session_state=f109bb89-cd34-4374-b084-c3c1cf2c8a0b&code=0aaca7b5-a314-4c07-8212-818cb4b7e8d0.f109bb89-cd34-4374-b084-c3c1cf2c8a0b.1dc15d06-d8b9-4f0f-a042-727eaa6b98f7`*

откуда берется  значение `code` для обмена на `token`

### Шаг 2. Получение token

`code` полученный на первом шаге меняется на токен

***POST*** */realms/SST/protocol/openid-connect/token*

**{red}(Заголовок)** `Content-Type: application/x-www-form-urlencoded`

#|
||

Параметр

|

Обязательный

|

Значение

|

Примечание

||
||

grant_type

|

да

|

authorization_code

|

Не изменяется. Иное значение п\*/realms/SST/protocol/openid-connect/auth\*риведет к ошибке запроса

||
||


|


|


|


||
||

client_id

|

да

|

client-backoffice

|

Идентифкатор приложения запрашивающего авторизацию, должно совпадать с идентификатором переданном в шаге 1

||
||


|


|


|


||
||

client_secret

|

да

|


|

Указан в настройках клиента (credentials) в админ. панели Keycloak

||
||


|


|


|


||
||

code

|

да

|


|

Значение полученное из шага 1

||
||


|


|


|


||
||

redirect_uri

|

да

|


|

Значение указанное в шаге 1

||
|#

Пример ответа

`{`
`"access_token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICItNUlsX2I0cUktdWFvaEI3d244UHY3WEM2UEktU3BNbmZCRnlJZUx6QTJNIn0.eyJleHAiOjE1OTIzNDM5NDEsImlhdCI6MTU5MjM0MzY0MSwiYXV0aF90aW1lIjoxNTkyMzQwODA1LCJqdGkiOiJlYjlhNTc2NS1jYmVhLTQ2ZWMtYTk4NS0wOWFkYTM5NTk5YjIiLCJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjgwODAvYXV0aC9yZWFsbXMvYXBwc2RldmVsb3BlcmJsb2ciLCJzdWIiOiIxZGRlM2ZjMy1jNmRiLTQ5ZmItOWIzZC03OTY0YzVjMDY4N2EiLCJ0eXAiOiJCZWFyZXIiLCJhenAiOiJwaG90by1hcHAtY29kZS1mbG93LWNsaWVudCIsInNlc3Npb25fc3RhdGUiOiJmMTA5YmI4OS1jZDM0LTQzNzQtYjA4NC1jM2MxY2YyYzhhMGIiLCJhY3IiOiIwIiwic2NvcGUiOiJwcm9maWxlIiwibmFtZSI6IkthcmdvcG9sb3YiLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJzZXJnZXkiLCJmYW1pbHlfbmFtZSI6IkthcmdvcG9sb3YifQ.KHCNF0Rn-I7iFosB3oEaWetRw9lhSkkP0-Ef6iW2GAZuuI-GQtZUBDAD_aEDtLTdUpvGL8MKx8Os0qbUZKJJhBhTAJyz2DycgY--ROc_vLbPtJSll-F68tHT6KgC2etbTjpz4Ira6PaLigkT80zGb6tpnQmm1o7a4IGQ40-faKC4fivdfblypGqgRnniOGXMLGpzO2Ln92w1azjFAyOCIBhe3Nlcofjupi26qNGrJKuwBudzZgZCla9RDWm2MUTqMW65AOUpOmiJCd5E4JxbwOuG6H2tbYI2Z-ajQXzzcodmCAWfWu2oRkMaAuNImph8W9tRrqCQ0wlb55tXnUvEuw",`
`"expires_in": 300,`
`"refresh_expires_in": 1800,`
`"refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJlYWQyMDZmOS05MzczLTQ1OTAtOGQ4OC03YWNkYmZjYTU5MmMifQ.eyJleHAiOjE1OTIzNDU0NDEsImlhdCI6MTU5MjM0MzY0MSwianRpIjoiOGE2NTdhMDktYTQ3My00OTAyLTk1MjItYWYxMGFkMzUwYzUyIiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwL2F1dGgvcmVhbG1zL2FwcHNkZXZlbG9wZXJibG9nIiwiYXVkIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwL2F1dGgvcmVhbG1zL2FwcHNkZXZlbG9wZXJibG9nIiwic3ViIjoiMWRkZTNmYzMtYzZkYi00OWZiLTliM2QtNzk2NGM1YzA2ODdhIiwidHlwIjoiUmVmcmVzaCIsImF6cCI6InBob3RvLWFwcC1jb2RlLWZsb3ctY2xpZW50Iiwic2Vzc2lvbl9zdGF0ZSI6ImYxMDliYjg5LWNkMzQtNDM3NC1iMDg0LWMzYzFjZjJjOGEwYiIsInNjb3BlIjoicHJvZmlsZSJ9.WevUHYd7DV3Ft7mFJnM2iLlArotBvLlMfQxlcy0nig8",`
`"token_type": "bearer",`
`"not-before-policy": 0,`
`"session_state": "f109bb89-cd34-4374-b084-c3c1cf2c8a0b",`
`"scope": "profile"`
`}`