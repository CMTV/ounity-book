# Доступ к языку

Мы спроектировали класс `Language` так, что невозможно изменить его из кода. Но иногда это все же надо сделать. Вот несколько причин:

* Редактор языка, которым мы займемся далее, должен менять язык из кода
* Кто-то из пользователей хочет написать дополнение к нашему пакету. В этом случае ему тоже может потребоваться добавлять/удалять фразы изнутри скриптов

## Основа

Для прямого доступа к языку создадим отдельный класс `LanguageAccess` по пути `Scripts/Editor`.

```csharp
using System;
using System.Collections.Generic;
using System.Reflection;
using UnityEditor;

namespace I18n.EditorUtilities
{
    public class LanguageAccess
    {
        Language lang;

        public Language Lang => lang;

        public LanguageAccess(Language lang)
        {
            this.lang = lang;
            EditorUtility.SetDirty(lang);
        }
    }
}
```

Тут все достаточно просто, за исключением вызова метода `SetDirty(lang)`. Зачем он нужен? Этим вызовом мы говорим Unity чтобы он действительно записывал изменения в языке. Без вызова `SetDirty` все изменения будут потеряны после перезапуска Unity.

## Список фраз

Добавим в наш класс то, ради чего он и создавался — публичную переменную, ссылающуюся на список фраз языка.

```csharp
public List<Phrase> phrases;
```

Для того, чтобы ее инициализировать, нам потребуется еще одно свойство:

```csharp
FieldInfo Field
{
    get
    {
        return typeof(Language).GetField("phrases", BindingFlags.NonPublic | BindingFlags.Instance);
    }
}
```

Здесь мы получаем данные о приватной переменной `phrases` класса `Language`.

Добавим инциализацию нашего списка фраз в конце конструктора. В этом нам поможет рефлексия!

```csharp hl_lines="6"
public LanguageAccess(Language lang)
{
    this.lang = lang;
    EditorUtility.SetDirty(lang);

    phrases = (List<Phrase>)Field.GetValue(lang);
}
```

Осталось только написать метод `Save()`, который применяет заменяет старый список фраз языка списком из нашего класса:

```csharp
public void Save()
{
    Field.SetValue(lang, phrases);
}
```

## Использование

Приведу небольшой пример, как можно добавлять и удалять фразы языка с помощью нашего класса:

```csharp
var _lang = new LanguageAccess(lang);

_lang.phrases.Add(new Phrase("new_id", "new text"));
_lang.phrases.RemoveAt(2);

_lang.Save();
```

В этом примере мы добавляем новую фразу, а также удаляем фразу с индексом 2.

## Итог

У нас есть класс `Language`, который невозможно случайно изменить, а также класс `LanguageAccess`, который нужен, если действительно нужно что-то изменить в языке напрямую. Очень удобно!

Полный код класса `LanguageAccess`:

```csharp
using System;
using System.Collections.Generic;
using System.Reflection;
using UnityEditor;

namespace I18n.EditorUtilities
{
    public class LanguageAccess
    {
        Language lang;

        public Language Lang => lang;

        public List<Phrase> phrases;

        FieldInfo Field
        {
            get
            {
                return typeof(Language).GetField("phrases", BindingFlags.NonPublic | BindingFlags.Instance);
            }
        }

        public LanguageAccess(Language lang)
        {
            this.lang = lang;
            EditorUtility.SetDirty(lang);

            phrases = (List<Phrase>)Field.GetValue(lang);
        }

        public void Save()
        {
            Field.SetValue(lang, phrases);
        }
    }
}
```