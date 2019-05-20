# Язык

Пора создать базовые классы и структуры, составляющие фундамент пакета.

## Структура "Информация о языке"

Нам нужно как-то различать один язык от другого. Какие параметры имеет любой язык? Его название ("Русский") и код ("ru").

Создадим структуру `LanguageInfo` в папке `Scripts/Runtime`, которая хранит эти две переменные. Почему структура? Потому что она требует меньше ресурсов, а передавать ее по ссылке мы все равно не будем.

```csharp
using System;
using UnityEngine;

namespace I18n
{
    [Serializable]
    public struct LanguageInfo
    {
        [SerializeField]
        string code, name;

        public LanguageInfo(string code, string name)
        {
            this.code = code;
            this.name = name;
        }

        public string Code => code.Trim();
        public string Name => name.Trim();
    }
}
```

Обратите внимание на то, что наша структура помечена атрибутом `[Serializable]`. Атрибут нужен потому что в будущем мы будем хранить объекты этой структуры в `ScriptableObject`, а он сохраняет только сериализуемые объекты. Другими словами, если этот атрибут убрать, то после каждого перезапуска Unity наш язык будет забывать, кто он и как его зовут.

Также стоит отметить атрибут `[SerializeField]` с его помощью мы можем редактировать приватные переменные структуры внутри Unity, но не можем сделать это напрямую из кода. Это сделано намеренно. Нам не нужно, чтобы у других программистов была возможность изменить информацию о языке каким либо образом кроме редактора Unity.

Единственное, что мы позволяем делать изнутри кода — читать значения переменных. Для этого создаем два публичных `get` свойства под каждую переменную:

```csharp
public string Code => code;
public string Name => name;
```

## Структура "Фраза"

Вспомните раздел "[Как использовать I18n](../about/index.md#как-использовать-i18n)". Там мы используем метод `Phrase("game_over")`. Но что на самом деле происходит в момент вызова этого метода?

Любой язык в I18n состоит из набора фраз. Фраза — две переменные: идентификатор фразы и ее текст. Например, у нас есть английский язык. В нем содержится фраза с идентификатором `game_over` и текстом `Game over`. Но есть и русский язык. Там тоже есть фраза с идентификатором `game_over`, но с другим текстом: `Конец игры`.

[![Структура языка](images/language-structure.png)](images/language-structure.png)

В результате вызова `Phrase("game_over")` мы получаем текст фразы нужного языка с данным идентификатором.

Создадим структуру `Phrase` в папке `Scripts/Runtime`, которая и будет представлять собой фразу.

```csharp
using System;
using UnityEngine;

namespace I18n
{
    [Serializable]
    public struct Phrase
    {
        [SerializeField]
        string id, text;

        public Phrase(string id, string text)
        {
            this.id = id;
            this.text = text;
        }

        public string Id => id;
        public string Text => text;
    }
}
```

Все комментарии к коду из предыдущего раздела работают и для этой структуры.

## Класс "Язык"

Пришло время самих языков. Как мы уже выяснили выше, язык представляет собой лишь набор фраз и информацию о самом себе (название и код). Эта информация никак не зависит от сцен в игре. Поэтому выгоднее всего будет создать язык в виде скриптуемого объекта `ScriptableObject`.

Создадим класс `Language` в папке `Scripts/Runtime`:

```csharp
using System.Collections.Generic;
using UnityEngine;

namespace I18n
{
    [CreateAssetMenu(fileName = "New Language", menuName = "I18n/Language", order = 0)]
    public class Language : ScriptableObject
    {
        [SerializeField]
        bool isDefault;

        [SerializeField]
        LanguageInfo info;

        [SerializeField]
        List<Phrase> phrases;
    }
}
```

Обратите внимание на то, что все три переменные класса приватные. Это нужно для того, чтобы какой-нибудь другой программист случайно не изменил их. Любое изменение этих перменных должно происходить только через интерфейс редактора Unity! Для этого все эти переменные помечены атрибутом `[SerializeField]`.

Уже сейчас вы можете создать несколько языковых ассетов. Для где-нибудь внутри проекта кликните правой кнопкой и выберите "Create > I18n > Language".

[![Создание нового языка](images/create-language.png)](images/create-language.png)

Выделите созданный язык. В инпекторе вы можете задать код языка (например, `ru`) и его название (например, `Русский`).

[![Стандартный инспектор языка](images/native-language-inspector.png){: .w7 }](images/native-language-inspector.png)

Не расслабляйтесь! Мы только начали работу над классом `Language`. Еще много чего надо сделать.

### Язык по умолчанию

Мы можем пометить язык, как "Язык по умолчанию". Если не выбрано другого языка, то будет выбран данный язык. Создадим публичное `get` свойства для получения значения переменной `isDefault`.

```csharp
public bool IsDefault => isDefault;
```

### Информация о языке

Сейчас нет никакой возможности получить информацию о языке изнутри кода. Исправим это путем добавления публичного `get` свойства, которое возвращает переменную `info`:

```csharp
public LanguageInfo Info => info;
```

Напомню еще раз, почему мы не можем просто сделать `info` публичной. Если она будет публичной, какой-нибудь невнимательный программист, который использует ваш пакет, может по ошибке внутри своего ~~говно~~ кода изменить информацию о языке. Это может привести к страшным последствиям. Просто подумайте о том, что произойдет, если у код русского языка (`ru`) внезапно станет равен `en`!

### Получение всех фраз

Частно нам будет нужно работать со всем списком фраз одновременно. Создадим публичное `get` свойство `Phrases`, возвращающее список фраз в режим только для чтения (`IReadOnlyCollection`):

```csharp
public IReadOnlyCollection<Phrase> Phrases => phrases.AsReadOnly();
```

### Работа с фразами

Пора дать другим людям возможность получать текст фразы из языка по ее идентификатору.

Стоит уточнить, что идентификатор может иметь определенные знаки в конце: `.`, `?`, `!`, `:`, `...`. Если хоть один из них присутствует в конце ID, мы добавим его в конце текста фразы. Зачем? Пусть у нас есть нейтральная фраза с ID `game_over` и текстом `Конец игры`. Но что если мы хотим получить "Конец игры?" или "Конец игры!". Создавать под каждый вариант свою фразу? Неудобно!

А вот с проверкой на пунктуацию в конце ID мы можем просто написать: `GetPhrase("game_over?")` или `GetPhrase("game_over!")` и получить в конце нужный знак.

Поэтому, сначала напишем методы, которые отделяют знаки пунктуации от ID:

```csharp
public string[] SplitId(string id)
{
    var punctuation = new string[]
    {
        "...", ".", ":", ";",
        "!?", "?!", "!", "?"
    };

    string ending = "";

    foreach (var item in punctuation)
    {
        if (id.EndsWith(item))
        {
            ending = item;
            break;
        }
    }

    var clearId = ending != "" ? id.Substring(0, id.LastIndexOf(ending)) : id;

    return new string[] { clearId, ending };
}

public string GetId(string id)
{
    return SplitId(id)[0];
}

public string GetEnding(string id)
{
    return SplitId(id)[1];
}
```

Все действие происходит в первом методе (`SplitId`). В нем мы разделяем ID на "чистый ID" (идентификатор фразы) и "концовку" — знак пунктуации из списка выше. Затем возвращаем и то и другое в виде массива из двух элементов.

Методы `GetId` и `GetEnding` являются сокращениями на случай, если нам нужен только ID или только концовка.

Теперь пора реализовать метод `GetPhrase` для получения текста фразы по ее ID и некоторые его вариации!

Сначала добавим библиотеку `System.Linq` в наш файл:

```csharp
using System.Linq;
```

Реализуем базовый вариант `GetPhrase`, который принимает только ID фразы с необязательным параметром `strict`:

```csharp
public string GetPhrase(string id, bool strict = false)
{
    var splitId = SplitId(id);

    string clearId = splitId[0];
    string ending = splitId[1];

    var matches = phrases.Where(phrase => phrase.Id == clearId);

    if (matches.Count() == 0)
    {
        return strict ? null : id;
    }

    return matches.First().Text + ending;
}
```

Этот метод состоит из двух частей. В первой мы разбиваем данный ID на "чистый ID" и концовку.

Во второй части метода мы ищем, есть ли в языке фразы с данным "чистым ID". Если есть, возвращаем текст фразы с приплюсованной концовкой.
Если нет, то в зависимости от параметра `strict` мы либо возвращаем ID, либо возвращаем `null`. Зачем возвращать `null`? Он нужен для того, чтобы точно понимать, что такой фразы в языке нет.

Теперь реализуем метод `GetPhrase`, принимающий параметры. Например, у нас есть вот такой текст фразы: `{killer} убил тебя`. В этой фразе вместо `{killer}` должен быть никнейм убийцы. Этот самый ник надо передать в фразу в качестве параметра вместе с ID.

```csharp
public string GetPhrase(string id, Dictionary<string, string> phraseParams)
{
    string text = GetPhrase(id, strict: true);

    if (text == null)
    {
        return id;
    }

    foreach (var param in phraseParams)
    {
        text = text.Replace("{" + param.Key + "}", param.Value);
    }

    return text;
}
```

Тут все просто. Проверяем, если ли вообще такая фраза. Если есть, то получаем ее текст и заменяем все выражения вида `{...}` на нужные строки. Если фразы нет, возвращаем ID.

Кстати, как раз тут нам и пригодился параметр `strict`. С его помощью мы сразу понимаем, что искомой фразы нет и возвращаем ее ID.

### Итоговый код

После всех модификаций, у нас получается следующий класс `Language`:

```csharp
using System.Linq;
using System.Collections.Generic;
using UnityEngine;

namespace I18n
{
    [CreateAssetMenu(fileName = "New Language", menuName = "I18n/Language", order = 0)]
    public class Language : ScriptableObject 
    {
        [SerializeField]
        bool isDefault;

        [SerializeField]
        LanguageInfo info;

        [SerializeField]
        List<Phrase> phrases;

        public bool IsDefault => isDefault;

        public LanguageInfo Info => info;

        public IReadOnlyCollection<Phrase> Phrases => phrases.AsReadOnly();

        #region GetPhrase

        public string GetPhrase(string id, bool strict = false)
        {
            var splitId = SplitId(id);

            string clearId = splitId[0];
            string ending = splitId[1];

            var matches = phrases.Where(phrase => phrase.Id == clearId);

            if (matches.Count() == 0)
            {
                return strict ? null : id;
            }

            return matches.First().Text + ending;
        }

        public string GetPhrase(string id, Dictionary<string, string> phraseParams)
        {
            string text = GetPhrase(id, strict: true);

            if (text == null)
            {
                return id;
            }

            foreach (var param in phraseParams)
            {
                text.Replace("{" + param.Key + "}", param.Value);
            }

            return text;
        }

        #endregion

        public string[] SplitId(string id)
        {
            var punctuation = new string[]
            {
                "...", ".", ":", ";",
                "!?", "?!", "!", "?"
            };

            string ending = "";

            foreach (var item in punctuation)
            {
                if (id.EndsWith(item))
                {
                    ending = item;
                    break;
                }
            }

            var clearId = ending != "" ? id.Substring(0, id.LastIndexOf(ending)) : id;

            return new string[] { clearId, ending };
        }

        public string GetId(string id)
        {
            return SplitId(id)[0];
        }

        public string GetEnding(string id)
        {
            return SplitId(id)[1];
        }
    }
}
```