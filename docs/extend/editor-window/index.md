description: Подробные инструкции по созданию собственного окна редактора, пункта в меню, назначению горячих клавиш и иконки, а также множество других полезных данных.

# Окно редактора

Весь редактор Unity состоит из набора окон: Сцена, Проект, Инспектор, Иерархия объектов и так далее... Стандартные окна помогают практически всегда. Но все же часто приходится создавать свои собственные — для выполнения специфичных задач.

В этой статье приведена вся информация по созданию и настройке собственного окна. От C# класса, до контекстного меню и иконки перед названием.

## Основа окна

Так как мы расширяем редактор Unity, в проекте обязательно должна быть папка с названием `Editor`. Внутри этой папки (или какой-то подпапки) создайте C# класс окна. Например, `TestWindow.cs`.

![Демонстрация файла TestWindow.cs](images/test-window-class.png)

Для начала нужно подключить пространства имен `UnityEngine` и `UnityEditor`. Внутри последнего есть класс `EditorWindow`, от которого наследуют все окна редактора, в том числе и наше окно `TestWindow`:

```csharp
using UnityEngine;
using UnityEditor;

public class TestWindow : EditorWindow
{

}
```

Теперь создадим функцию `ShowWindow`, которая будет задавать начальные параметры окна: название, размеры и другие.

```csharp
using UnityEngine;
using UnityEditor;

public class TestWindow : EditorWindow
{
    public static TestWindow ShowWindow()
    {
        TestWindow window = GetWindow<TestWindow>();

        // Устанавливаем название окна
        window.titleContent = new GUIContent("Test Window");

        // Устанавливаем мин. и макс. размеры
        window.minSize = new Vector2(150, 150);
        window.maxSize = new Vector2(1000, 500);

        return window;
    }
}
```

!!! заметка
    Название функции, ее уровень доступа и тип возвращаемого значения можно выбрать любыми. Я указал именно такие, так как считаю, что с ними работать гораздо удобнее. Настоятельно рекомедную использовать именно их.

## Пункт в меню

Создадим пункт для вызова окна в основном меню Unity. Подобные пункты создаются с помощью атрибута `MenuItem`, который нужно расположить **перед** методом `ShowWindow`. В скобках указывается местоположение пункта.

```csharp hl_lines="6"
using UnityEditor;
using UnityEngine;

public class TestWindow : EditorWindow
{
    [MenuItem("Window/Test Window")]
    public static TestWindow ShowWindow()
    {
        TestWindow window = GetWindow<TestWindow>();

        // Устанавливаем название окна
        window.titleContent = new GUIContent("Test Window");

        // Устанавливаем мин. и макс. размеры
        window.minSize = new Vector2(150, 150);
        window.maxSize = new Vector2(1000, 500);

        return window;
    }
}
```

!!! важно
    С помощью атрибута `MenuItem` невозможно создать пункт прямо в главном меню Unity. Код ниже приведет к ошибке:

    ```csharp
    [MenuItem("Test Window")]
    ```

    Пункт меню обязательно должен иметь родителя, например, "Window" или что-то собственное.

Так как в скобках мы указали `Window/Test Window`, то пункт `Test Window` появится в списке меню `Window`.

![Пункт в меню](images/menu-item.png){ .border }

Если указать `Lol/Boom`, то в верхней панели Unity будет создан список `Lol` с одним пунктом меню — `Boom`:

![Нестандартный пункт в меню](images/menu-item-custom.png){ .border }

## Вызов окна из кода

Для отображения окна нужно просто вызвать метод `ShowWindow`:

```csharp
// ...

TestWindow.ShowWindow();

// ...
```

Бывают случаи, когда нужно изменить некоторые параметры окна во время его создания. Код ниже показывает наше окно с измененным названием:

```csharp
// ...

var window = TestWindow.ShowWindow();
window.titleContent = new GUIContent("Новое название!");

// ...
```

Естественно, обращаясь к переменной `window`, менять можно не только название, но и все остальное (размеры, иконку и т.д.).

## Интерфейс окна

Если два принципиально разных подхода к созданию интерфейса:

* [UI Elements](../../ui/ui-elements/about/index.md) — для продвинутых и сложных интерфейсов
* IMGUI — для простеньких интерфейсов

Подробнее об обеих технологиях читайте в соответствующих разделах учебника. Ниже приведены примеры одинаковых интерфейсов на основе обоих вариантов.

### UI Elements

У любого окна есть переменная `rootVisualElement`, которая является корневым элементом окна. В этот корневой элемент можно добавлять различные элементы интерфейса, прицеплять таблицы стилей. Лучше всего это делать в методе `OnEnable`.

```csharp
void OnEnable()
{
    // Текстовое поле
    var textField = new TextField("Текстовое поле");
    rootVisualElement.Add(textField);

    // Поле выбора цвета
    var colorField = new ColorField("Цвет текста");
    colorField.showAlpha = false;
    rootVisualElement.Add(colorField);

    //
    // Кнопка "Отправить"
    //

    var applyButton = new Button(() => 
    {
        Debug.Log(
            "<color='#" + ColorUtility.ToHtmlStringRGB(colorField.value) + "'>" 
            + textField.value + 
            "</color>"
        );
    });

    applyButton.text = "Отправить";
    rootVisualElement.Add(applyButton);
}
```

![Интерфейс на основе UI Elements](images/interface-uielements.png){ .border }

### IMGUI

Отрисовку интерфейса с помощью IMGUI нужно выполнять в методе `OnGUI`:

```csharp
string text;
Color color;

void OnGUI()
{
    // Текстовое поле
    text = EditorGUILayout.TextField("Текстовое поле", text);
    
    // Поле выбора цвета
    color = EditorGUILayout.ColorField(new GUIContent("Цвет текста"), color, true, false, false);
    
    // Кнопка "Отправить"
    if (GUI.Button(new Rect(10, 50, 150 ,30), "Отправить"))
    {
        Debug.Log(
            "<color='#" + ColorUtility.ToHtmlStringRGB(color) + "'>" 
            + text + 
            "</color>"
        );
    }
}
```

![Интерфейс на основе IMGUI](images/interface-imgui.png){ .border }

## Горячая клавиша

Горячую клавишу для окна можно назначить в конце атрибута `MenuItem` через пробел для отделения от местоположения пункта меню.

Пример горячей клавиши, которая открывает наше окно по нажатию на "Shift + G":

```csharp
[MenuItem("... #g")]
```

Под троеточием скрывается путь к пункту меню. Он нам не нужен. Сама горячая клавиша определяется так: `#t`. `#` — специальный символ, обозначающий клавишу Shift. Список доступных символов:

* `#` — Shift
* `&` — Alt
* `LEFT`, `RIGHT`, `UP`, `DOWN` — клавиши стрелок
* `F1...F12` — клавиши от F1 до F12
* `HOME`, `END`, `PGUP`, `PGDN`

Если горячей клавишей является какая-то обычная буквенная клавиша, то перед ней нужно поставить `_`. Пример открытия окна по клавише `G`:

```csharp
[MenuItem("... _g")]
```

## Иконка окна

TODO

## Контекстное меню окна

TODO