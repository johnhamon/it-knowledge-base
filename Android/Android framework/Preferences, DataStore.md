#### Preferences
SharedPreferences - Интерфейс для доступа и модификации данных о пользовательской информации, возвращаемых [Context.getSharedPreferences(String, int)](https://developer.android.com/reference/android/content/Context#getSharedPreferences(java.lang.String,%20int)). Для любого конкретного набора предпочтений существует единственный экземпляр этого класса, который совместно используется всеми клиентами. Изменения в предпочтениях должны проходить через [Editor](https://developer.android.com/reference/android/content/SharedPreferences.Editor), чтобы значения предпочтений оставались в согласованном состоянии и контролировали момент их фиксации в памяти. Объекты, возвращаемые различными методами `get`, должны рассматриваться приложением как неизменяемые.

API `SharedPreferences` поддерживает чтение и запись простых пар ключ-значение **из файла, который переживает пользовательские сессии**. Библиотека `Preference` использует частный экземпляр `SharedPreferences`, так что доступ к нему может получить только ваше приложение.

Вызываются `Preferences` из объекта `Activity`:
```Kotlin
val sharedPref = activity?.getPreferences(Context.MODE_PRIVATE) ?: return
with (sharedPref.edit()) {
	putInt(getString(R.string.saved_high_score_key), newHighScore)    
	apply()  
}
```
Note: Примечание: Этот класс обеспечивает сильные гарантии согласованности. Он использует дорогостоящие операции, которые могут замедлить работу приложения. Часто меняющиеся свойства или свойства, где потери могут быть допустимы, должны использовать другие механизмы

Класс не поддерживает использование в нескольких процессах
#### DataStore
Позволяет хранить пары ключ-значение или типизированные объекты с помощью [протокольных буферов](https://protobuf.dev/). DataStore использует корутины Kotlin и Flow для асинхронного, последовательного и транзакционного хранения данных.

`DataStore` предоставляет две различные реализации: **Preferences DataStore** и **Proto DataStore**.

- **Preferences DataStore** хранит и получает доступ к данным с помощью ключей. Эта реализация не требует предопределенной схемы и не обеспечивает безопасность типов.
- **Proto DataStore** хранит данные как экземпляры пользовательского типа данных. Эта реализация требует определения схемы с использованием Protobuf, но обеспечивает безопасность типов.

Никогда не создавайте более одного экземпляра `DataStore` для данного файла в одном и том же процессе. Это может привести к нарушению функциональности `DataStore`. Если в одном и том же процессе для данного файла активно несколько `DataStore`, то при чтении или обновлении данных `DataStore` будет выбрасывать ошибку IllegalStateException.

Общий тип `DataStore` должен быть неизменяемым. Мутирование типа, используемого в `DataStore`, сводит на нет все гарантии, которые предоставляет `DataStore`, и создает потенциально серьезные, трудноустранимые ошибки. Настоятельно рекомендуется использовать протокольные буферы, которые обеспечивают гарантии неизменяемости, простой API и эффективную сериализацию.

Никогда не смешивайте использование `SingleProcessDataStore` и `MultiProcessDataStore` для одного и того же файла. Если вы собираетесь обращаться к DataStore из нескольких процессов, всегда используйте `MultiProcessDataStore`.

Инстанцируется вот так: 
```Kotlin
val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "settings")
```
`DataStore` - асинхронный API, поэтому использовать в блокирующем коде нужно специальным образом средствами языка/фреймворка

https://developer.android.com/develop/ui/views/components/settings/use-saved-values
https://developer.android.com/reference/android/content/SharedPreferences
https://developer.android.com/topic/libraries/architecture/datastore
