
# Ова е exported build од проектот. За изворниот код, вклучувајќи ги сите скрипти, анимации, sprites и prefabs може да отидете на следниот [линк](https://drive.google.com/drive/folders/1fEO_eH30HPZtKqzLuTuwmLxpTQoNRDXX?usp=sharing)

# Retail Simulator

"Retail Simulator" е кратка top-down 2D видео игра развиена во Unity како семинарски проект по предметот "Визуелно Програмирање" во Летниот семестар 2024 година. Целта е да се достигне доволно поени пред да истече време (крајот на смената на работникот).

## Структура

За да се објасне структурата на играта, прво треба да се поминат неколку основни концепти во Unity алатката за создавање видео игри. \
\
Прво, секоја "стаза" во Unity се нарекува **сцена**. \
Се што е генерирано на екранот се генерира преку контејнери или т.н. **GameObjects**. \
Секој **GameObject** се состои од повеќе компоненти, како на пример Rigidbody2D и Box Collider 2D за овозможување физика, или Sprite Renderer за генерирање слика. \
**GameObject** контејнерите можат да се создаваат статички во Unity Editor, или да се генерираат "at runtime". \
Дополнително, во останатиот дел од документацијата **GameObjects** ќе ги нарекуваме "објекти". \
Ова е овозможено преку тоа што можеме било кој GameObject да го зачуваме како шема (Ова се нарекуваат **prefabs** или prefabricated објекти). \
Освен тоа, има некои специјални објекти како UI GameObjects или камерата. 


## Прва сцена - Main Menu Scene

Кога играчот ќе ја пушти играта за прв пат, ќе се најде прво на Main Menu сцената. Ова е едноставна сцена кадешто сите GameObjects се наоѓаат на т.н. "**Canvas**". Во канвасот можат да постојат само UI GameObjects. \
Освен претходно наведените компоненти, можеби најважната компонента која може да биде додадена на GameObject контејнер се скриптите. \
Скриптите одредуваат логичко однесување на секој GameObject и сите се пишуваат во C# програмскиот јазик.
Во почетната сцена имаме само една скрипта, "Main Menu Script.cs", која се состои од следниот код:

```c#
public class MainMenuScript : MonoBehaviour
{
    public GameObject TutorialImage;
    private bool TutorialImageActive = false;

    public void Start()
    {
        Application.targetFrameRate = 60;
    }

    // Start is called before the first frame update
    public void PlayGame()
    {
        SceneManager.LoadScene("MainShopScene", LoadSceneMode.Single);
    }

    public void QuitGame()
    {
        Application.Quit();
    }

    public void Update()
    {
        if (Input.GetKeyDown(KeyCode.H) && TutorialImageActive)
        {
            TutorialImage.SetActive(false);
            TutorialImageActive = false;
        } else if (Input.GetKeyDown(KeyCode.H) && !TutorialImageActive)
        {
            TutorialImage.SetActive(true);
            TutorialImageActive = true;
        }
    }

    public void BringUpTutorialImage()
    {
        TutorialImage.SetActive(true);
    }
}
```


Секој **GameObject** може да се наоѓа во една од две состојби - активна или неактивна. При кликање на копчето "Tutorial", 
едноставно ја менуваме состојбата на Tutorial UI сликата од неактивна во активна и обратно. Истото се поддржува преку стискање на копчето H на тастатура.
Понатаму се употребува техника кадешто некој дел од UI е автоматски скриен (во случај на смени при играње, на пр. играчот ја заврши активностата или сл.) и понатаму покажан на играчот кога потребно.
Ова го правиме преку тоа што прво ја наоѓаме сликата "at runtime" и автоматски ја криеме. Пример:

```c#
private void FindUIObjects()
{
    var TextObjects = GameObject.FindGameObjectsWithTag("UITag");
    var GameOverImageLocalVariable = GameObject.FindGameObjectsWithTag("GameOverImage");

    GameOverImage = GameOverImageLocalVariable[0];
    GameOverImage.SetActive(false);
    ScoreText = TextObjects[0].GetComponent<TextMeshProUGUI>();
    ShiftTimeText = TextObjects[2].GetComponent<TextMeshProUGUI>();
    GoalAmtText = TextObjects[1].GetComponent<TextMeshProUGUI>();
    //text object named "ShiftTime" is consistently found AFTER the text object named "GoalTime"
}
```

Функцијата е од скриптата "GameStateManagerScript.cs" којашто е behavior скрипта за GameStateManager објектот - овој го употребуваме понатаму за пренесување информации меѓу сцени, како и контролирање на состојбата на играта. \
Исто така можеме да соочиме дека секој script може да содржи референци кон други скрипти и објекти.

 


## Втора сцена - Main Shop Scene

Играчот во оваа сцена ја контролира камерата, т.е камерата го следи играчот, кој што се контролира преку WASD копчињата.  
Тука се достапни две активности: **Falling Fruits** и **Scanning Items**. На подот е обележано каде треба да застане играчот за да започне активноста. 

Player објектот којшто играчот го контролира е целосно анимиран благодарение на новиот Input System на Unity како и Animator компонентата. Анимациите се создаваат "in-app" и InputManager објектот контролира која ќе се одигра.

Дополнително е значително да се спомне дека сите објекти се наоѓаат на некојси слој (Layer). Објектите во одреден слој исто така имаат редослед на цртање, пришто тие со повисока вредност во редоследот се цртаат **над** оние со пониска. 
Бидејќи можеме да се движиме во четири насоки во играта, потребно е динамички да одредиме кој ќе биде редоследот на нашите објекти. Тоа е овозможено преку кодот:

```c#
SpriteRenderer spriteRenderer = GetComponent<SpriteRenderer>();
spriteRenderer.sortingOrder = Mathf.RoundToInt(transform.position.y * -100);
```

Falling Fruits активностата може да се започне кога сака играчот, додека за Scanning Items е потребно да се почека да дојде муштерија. Муштеријата доаѓа по некое време, а самиот NPC објект се бира по случаен избор да се инстанцира. \
Сите NPC prefabs се сместени во хиерархија, од која што GameStateManagerScript.cs бира еден да создаде. 

```c#
    private void SpawnCustomer()
    {
        var Customer = CustomerPrefabs[Random.Next(0, CustomerPrefabs.Length)];

        if (CustomerSpawnArea == null)
        {
            CustomerSpawnArea = GameObject.FindGameObjectsWithTag("CustomerSpawnArea")[0];
        }

        Customer.transform.position = CustomerSpawnArea.transform.position;
        Instantiate(Customer);
        
    }
```

Се случува при reloading на сцена да се изгубат сите поврзани компоненти референцирани во скриптата. Поради тоа на повеќе места низ скриптите кога ќе се обидеме да референцираме друг објект е потребно да провериме дали во соодветниот момент
објектот е null, и ако е, да го побараме пак.

За двете активности прво се појавува message prompt која го прашува играчот дали би сакал/а да продолжи. \

Во оваа сцена ја менаџираме состојбата на играта. Смената на работникот (играчот) трае 12 саати - за таа причина за играчот е достапен тајмер којшто претставува време на денот во секој момент. \
Истиот тајмер се ажурира на секои 5 секунди.






## Трета сцена - Falling Fruits

Оваа сцена е едноставна игра каде што контролираме кошница со која што собираме овошје кое што паѓа од горе. Постојат 3 вида на овошје: \
Лимон, домат и портокал.


Сите три носат различен број на поени. Портокалот носи 100 поени за играчот, доматот 200 додека лимонот одзима 100 поени. 
Во сцената постои апстрактен објект којшто ги инстанцира овие објекти над камерата на секои 2 секунди, при што случајно избира каков тип на овошје ќе инстанцира (Ова го прави преку т.н. prefabs). 

```c#
private void SpawnObject()
{
    Vector2 spawnPos = new Vector2();
    float xPos = rand.Next(-7, 7);
    spawnPos.Set(xPos, transform.position.y);

    var WhichFallingObjectToSpawn = FallingObjects[rand.Next(0, FallingObjects.Length)];

    Instantiate(WhichFallingObjectToSpawn, spawnPos, transform.rotation);
}
```

Оваа активност трае точно 20 секунди.

## Четврта сцена - Scanning Items

Последната сцена не е достапна се додека не дојде муштерија до шалтерот. Секој муштерија има различна комбинација на производи кои што сака да ги купи - сите овие се одбрани по случаен избор од сцената "Main Shop Scene" и пренесени на SceneStateManager
објектот. 
Играчот треба соодветно да одбере во која обоена коцка треба да отиде соодветниот производ. \
Беше планирано поголема селекција на производи, но само неколку се достапни во играта. \
Генерално по ова правило се водат производите: \
Храна оди во црвената коцка. \
Пијалаци одат во сината коцка. \
Останато оди во зелената коцка. 

Самите објекти се создаваат "at runtime" преку соодветен "prefab". Овие објекти се во хиерархија и секој производ е варијанта на родителот објект "GenericAreaItem". 

Оваа активност нема временско ограничување. 


## Други забелешки и коментари

1. Играта е прелесна. Всушност не е возможно да се изгуби.
2. Колизии на **Scanning Items** сцената се проблематични бидејќи директно менуваме позиција на самиот објект а не пренесуваме информации на Rigidbody преку којшто се овозможува физиката.


## Credits

[Song](https://www.youtube.com/watch?v=l7SwiFWOQqM)


