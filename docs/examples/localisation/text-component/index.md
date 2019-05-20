# Текстовый компонент

Наш пакет завершен на 97%. Все классы написаны, все функции реализованы. Но можно сделать еще кое-что. Чаще всего текст выводится с помощью компонента `Text` в пространстве имен `UnityEngine.UI`.

## Компонент LangText

Создадим класс `LangText` по пути `Scripts/Runtime`:

```csharp
using UnityEngine;
using UnityEngine.UI;

namespace I18n
{
    [AddComponentMenu("UI/Language Text")]
    [RequireComponent(typeof(Text))]
    public class LangText : MonoBehaviour
    {
        public bool updateOnLangChange = true;

        string phraseId;

        public Text Text
        {
            get
            {
                return GetComponent<Text>();
            }
        }

        void Start()
        {
            phraseId = Text.text;

            UpdateText();

            if (updateOnLangChange)
            {
                Lang.Instance.onLanguageSwitch.AddListener(UpdateText);
            }
        }

        public void UpdateText()
        {
            Text.text = Lang.Phrase(phraseId);
        }
    }
}
```

Разберем этот несложный код.

У нас есть переменная `updateOnLangChange`, которая определяет, будет ли обновлен текст фразы при смене языка.

Далее идет переменная `phraseId`, в которой хранится идентификатор фразы.

Затем идет публичное `get` свойство, которое возвращает компонент `Text`:

```csharp
public Text Text
{
    get
    {
        return GetComponent<Text>();
    }
}
```

В самом низу класса имеется функция `UpdateText`, которая получает текст фразы по ID и устанавливает текст UI элемента:

```csharp
public void UpdateText()
{
    Text.text = Lang.Phrase(phraseId);
}
```

Наконец, есть метод `Start`:

```csharp
void Start()
{
    phraseId = Text.text;

    UpdateText();

    if (updateOnLangChange)
    {
        Lang.Instance.onLanguageSwitch.AddListener(UpdateText);
    }
}
```

В нем мы сначала сохраняем ID фразы, чтобы его не потерять. Затем устанавливаем нужный текст. Наконец, если `updateOnLangChange` равен `true`, то мы слушаем событие `onLangaugeSwitch` и обновляем текст вновь, если требуется.