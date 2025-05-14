# C# Coding Guidelines

## Kapitel 1: Namngivningskonventioner

En snabb referens för hur olika typer av identifierare ska namnges i C#-kod.

| Typ                            | Konvention     | Exempel                                    | Kommentar |
|--------------------------------|----------------|--------------------------------------------|-----------|
| **Klasser**                    | PascalCase     | `CustomerService`                          |            |
| **Interfaces**                 | PascalCase + I | `IUserRepository`                          | `I` som prefix enligt .NET-standard |
| **Structs**                    | PascalCase     | `Point2D`                                  |            |
| **Publika/Interna metoder**    | PascalCase     | `CalculateTotal()`                         |            |
| **Privata metoder**            | PascalCase     | `ResetCache()`                             | Samma som publika för konsekvens |
| **Parametrar**                 | camelCase      | `string recipientAddress`                  |            |
| **Lokala variabler**           | camelCase      | `var orderCount = 0;`                      |            |
| **Privata fält (protected)**   | _camelCase     | `_userRepository`, `_logger`               | Inleds med `_` |
| **Konstanter**                 | PascalCase     | `MaxItems`, `DefaultTimeout`               | Undvik UPPER_CASE |
| **Testmetoder**                | Beskrivande    | `AddItem_WithValidItem_ShouldIncreaseCount()` | Format: `Method_State_ExpectedResult` |
| **Filnamn**                    | PascalCase     | `UserController.cs`                        | Ska spegla klassen |

---

## Kapitel 2: Kodstruktur och Layout

### 2.1 Rekommenderad ordning för klassmedlemmar

1. Konstanter och `readonly` fält  
2. Privata/protected fält  
3. **Konstruktorn – alltid överst** (inkl. primary constructor i C# 12)  
4. Publika properties  
5. Publika/internal metoder  
6. Privata/protected metoder (nära sina användare)

📌 Primary constructors i C# 12 är tillåtna och bör struktureras enligt samma principer.

📌 **Inversion of Control:** Injicera beroenden via konstruktorer. Undvik `new`.

#### Exempel
```csharp
public class OrderService(IOrderRepository orderRepository, ILogger<OrderService> logger)
{
    private readonly IOrderRepository _orderRepository = orderRepository;
    private readonly ILogger<OrderService> _logger = logger;

    public int OrderCount { get; private set; }

    public void ProcessOrder(Order order)
    {
        if (!Validate(order)) return;
        Save(order);
    }

    internal void PrintDebugInfo()
    {
        _logger.LogDebug("Order count: {Count}", OrderCount);
    }

    private bool Validate(Order order) => order != null && order.Items.Any();

    private void Save(Order order)
    {
        _orderRepository.Save(order);
        OrderCount++;
    }
}
```

---

### 2.2 Logisk gruppering enligt Clean Code

- Kod ska läsas uppifrån och ned.
- Hjälpmetoder direkt efter sina användare.
- Metoder som hör ihop bör grupperas.

---

### 2.3 Användning av `#region`

Använd sparsamt, bara där det förbättrar läsbarhet.  
Undvik en region per metod.

---

### 2.4 Access modifiers

Alltid specificera (även `private`).  
Undvik implicit synlighet.

---

### 2.5 Partial-klasser

Används endast när:
- Verktyg genererar kod.
- Klasser behöver delas upp av förståeliga skäl.

---

### 2.6 Tomma rader & indentering

- En tom rad mellan logiska block/metoder.  
- Använd 4 mellanslag (ej tabbar).

---

### 2.7 Metodlängd

- Kort, gör en sak.  
- Riktlinje: max 30–50 rader.

---

### 2.8 Användning av `this.`

- Använd endast där det behövs (t.ex. i konstruktör).  
- Annars undvik.

---

### 2.9 Initiering av samlingar

- Initiera till `[]` – samlingar ska inte vara `null`.

---

## Kapitel 3: Felhantering & Undantag

En bra strategi för felhantering bör vara:

- Konsekvent och lätt att förstå
- Tydlig utan att dölja fel
- Designad för tidig validering
- Loggande där kontext finns

Detaljer kring undantagstyper, custom exceptions och central hantering bör definieras efter behov.

### 3.1 Kasta inte undantag i onödan

```csharp
if (!int.TryParse(input, out var number))
{
    // Hantera ogiltigt värde utan att kasta
}
```

---

## Kapitel 4: Asynkron programmering (`async` / `await`)

### 4.1 Använd `async`/`await` konsekvent
```csharp
public async Task SaveAsync(Order order)
{
    await _repository.SaveAsync(order);
}
```

### 4.2 Namngiv `async`-metoder med `Async`-suffix  
```csharp
public async Task<List<Order>> GetOrdersAsync() { ... }
```

### 4.3 Undvik `async void`  
Undantag: event handlers.

### 4.4 Returnera `Task`, undvik `.Result`
```csharp
var result = await GetDataAsync(); // Bra
```

### 4.5 Använd `ConfigureAwait(false)` i bibliotek  
```csharp
await _httpClient.SendAsync(request).ConfigureAwait(false);
```

### 4.6 Parallellisering  
```csharp
await Task.WhenAll(method1Async(), method2Async());
```

### 4.7 Undantagshantering i async-kod  
Använd `try-catch` runt `await`.

### 4.8 Undvik async i konstruktor/property  
Använd t.ex. `InitializeAsync()` istället.

### 4.9 Använd `Task.Run` med försiktighet  
Endast för CPU-tunga operationer.

## Kapitel 5: Kodgranskning & Pull Requests

Att genomföra kodgranskningar är ett viktigt verktyg för att upprätthålla kodkvalitet, sprida kunskap och förhindra regressioner. En tydlig process hjälper alla att fokusera på rätt saker.

### 5.1 Syfte med kodgranskning

- Förbättra kodkvalitet och arkitektur
- Upptäcka buggar tidigt
- Säkerställa konventioner och riktlinjer följs
- Sprida kunskap i teamet

### 5.2 Vad som alltid ska kontrolleras

- ✅ Namngivning följer riktlinjerna (Kapitel 1)
- ✅ Kodstruktur och ordning (Kapitel 2)
- ✅ Felhantering är tydlig (Kapitel 3)
- ✅ Async/await används korrekt (Kapitel 4)
- ✅ Inga dolda beroenden skapas (IoC)
- ✅ Kod är testbar (enkel att enhetstesta)
- ✅ Samlingar är aldrig null
- ✅ Inga magiska värden eller hårdkodade strängar

### 5.3 Stil för kommentarer i PR

- **Saklig och respektfull ton**
- Kommentera på intention, inte person
- Ge gärna kodexempel vid förslag
- Föreslå förbättring, inte bara påpeka fel

### 5.4 Acceptanskriterier för merge

- All ny logik är testad
- Alla kommentarer är åtgärdade eller besvarade
- CI bygger och tester passerar
- Inga TODOs kvar utan kontext

📌 Det är teamets ansvar att säkerställa att kodgranskning inte bara är ett krav, utan en lärandeprocess.


## Kapitel 6: Testning och testbar kod

Testbar kod är en grundförutsättning för kvalitet, snabb utveckling och trygg refaktorering. Detta kapitel ger riktlinjer för hur man skriver kod som är enkel att enhetstesta och hur tester ska struktureras.

### 6.1 Grundprinciper

- Kod ska vara testbar utan beroende av miljö eller externa resurser.
- Beroenden ska injiceras, inte skapas inuti metoder/klasser.
- Små, separata metoder förenklar testning.

### 6.2 Teststruktur

- Använd `Arrange-Act-Assert`-mönstret i alla tester
- En testklass per testad klass
- En testmetod per logiskt scenario

```csharp
[Fact]
public void AddItem_WithValidItem_ShouldIncreaseCount()
{
    // Arrange
    var cart = new Cart();

    // Act
    cart.AddItem(new Item("Pen"));

    // Assert
    Assert.Single(cart.Items);
}
```

### 6.3 Namngivning av tester

- Format: `MethodName_StateUnderTest_ExpectedBehavior`
- Beskriv vad som testas, inte hur

Exempel: `Withdraw_WhenBalanceInsufficient_ShouldThrowException()`

### 6.4 Vad som bör testas

- Domänlogik
- Affärsregler
- Gränsfall och felhantering
- Tillstånd före/efter anrop

### 6.5 Vad som inte ska testas

- Implementation (t.ex. privata metoder)
- Tredjepartsbibliotek (mocka/stubba)
- UI-rendering eller logik som redan täcks av E2E-tester

### 6.6 Testbar kod i praktiken

**Dåligt exempel:**
```csharp
public class OrderService
{
    public void CreateOrder()
    {
        var db = new SqlConnection(...); // hårdkodat beroende
        db.Save(...);
    }
}
```

**Bättre:**
```csharp
public class OrderService
{
    private readonly IOrderRepository _repository;

    public OrderService(IOrderRepository repository)
    {
        _repository = repository;
    }

    public void CreateOrder(Order order)
    {
        _repository.Save(order);
    }
}
```

### 6.7 Verktyg och ramverk

- xUnit eller Nunit rekommenderas för enhetstester
- För Blazor-komponenttester rekommenderas [bUnit](https://bunit.dev)
- Vid användning av separat testdatabas kan [Respawn](https://github.com/jbogard/Respawn) användas för att tömma och återställa databasen
- För Blazor-komponenttester rekommenderas [bUnit](https://bunit.dev)
- FakeItEasy används för att skapa fakes/mocks på ett enkelt och läsbart sätt
- [Verify](https://github.com/VerifyTests/Verify) används för snapshot- och approval-tester, t.ex. JSON-resultat eller objektstrukturer

---

📌 Att skriva testbar kod handlar inte om tester – det handlar om **design**.

#### `.editorconfig`

- Se till att projektet innehåller en `.editorconfig`-fil som definierar konventioner.
- Editor- och byggverktyg som Visual Studio, Rider och dotnet-format kan då automatiskt upptäcka avvikelser.

**Exempel:**
```ini
# Namngivningskonventioner
[*.cs]
dotnet_naming_rule.private_fields_should_have_underscore.symbols = non_public_fields
non_public_fields.applicable_kinds = field
non_public_fields.applicable_accessibilities = private, protected
non_public_fields.required_modifiers = readonly

private_fields_should_have_underscore.naming_style = underscore_prefix
underscore_prefix.required_prefix = _
underscore_prefix.capitalization = camel_case
```

📌 Lägg till regler för spacing, newlines, brace styles etc. beroende på teamets preferenser.

#### ReSharper

- Använd ReSharper för att upptäcka, analysera och korrigera kodkvalitet.
- Skapa en gemensam **.DotSettings**-fil som delas i projektet.
- Inkludera regler för:
  - Naming conventions
  - File layout (ordning på medlemmar)
  - Formatteringsregler
  - Automatisk kodstädning

**Så här gör du:**
1. Gå till `ReSharper > Options > Code Editing > C# > Naming Style`
2. Definiera prefix `_` för privata fält
3. Skapa en "Code Cleanup"-profil och aktivera den vid save eller commit

📌 Exportera inställningar och versionera dem i projektet för att säkerställa att alla utvecklare har samma uppsättning.
