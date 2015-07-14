# Hilfe
### ich kann diese Klasse nicht testen!
<img src='images/help.gif' />

---
<div style='font-size:50%'>
  Kapitel 9
  <br />
  "Effektives Arbeiten mit Legacy Code"
  <br />
  Michael C. Feathers
</div>

---

# Problemfelder

1. Objekte der Klasse erstellen
2. Klasse im Test-Umfeld kompilieren
3. Konstruktor mit unerwünschten Nebeneffekten
4. Konstruktor hat Aufgaben, die überwacht werden müssen

---

<br />
### Fall 1:
## Irritierender Parameter
<img src='images/wtf.gif' />

---

### Fall 1: Irritierender Parameter

Beispiel:

``` cs
  public class CreditValidator {
    public CreditValidator(RatingSvcConnection connection,
                           CreditMaster creditMaster) {...}

    public bool IsCustomerCreditWorthy(Customer customer) {...}
  }
```

---

### Fall 1: Irritierender Parameter

Beispiel:

``` cs
  public class CreditValidator {
    public CreditValidator(RatingSvcConnection connection,
                           CreditMaster creditMaster) {...}

    public bool IsCustomerCreditWorthy(Customer customer) {...}
  }
```
``` cs
  public class RatingSvcConnection {
    public RatingSvcConnection(int port, string user, string pwd) {...}

    public void Connect() {...}
    public void Disconnect() {...}
    public void Retry() {...}
    public RatingReport ReportFor(int userId) {...}
  }
```

---

### Fall 1: Irritierender Parameter

Problem?

``` cs
  public class RatingSvcConnection {
    public RatingSvcConnection(int port, string user, string pwd) {...}

    public void Connect() {...}
    public void Disconnect() {...}
    public void Retry() {...}
    public RatingReport ReportFor(int userId) {...}
  }
```

* Serverzugriffe bei Testausführung!
* Langsam!
* Erreichbarkeit!

---

### Fall 1: Irritierender Parameter
Lösungsansatz
``` cs
  public interface IRatingSvcConnection {
    void Connect() {...}
    void Disconnect() {...}
    void Retry() {...}
    RatingReport ReportFor(int userId) {...}
  }
```
Schritt 1: Extract Interface


---

### Fall 1: Irritierender Parameter
Lösungsansatz
``` cs
  public FakeRatingServiceConnection: IRatingSvcConnection {
    public void Connect() { }
    public void Disconnect() { }
    public void Retry() { }
    public RatingReport ReportFor(int userId) { return null; }
  }
```
Schritt 2: Fake implementieren

---

### Fall 1: Irritierender Parameter
Lösungsansatz
``` cs
  void testNoSuccess() {
    var creditMaster = new CreditMaster();
    var ratingServiceConnection = new FakeRatingSvcConnection();
    var validator = new CreditValidator(ratingServiceConnection,
                                        creditMaster);
    var actual = validator.IsCustomerCreditWorthy(new Customer());
    Assert.That(actual, Is.True);
  }
```
Schritt 3: Fake im Test verwenden

---

### Fall 1: Irritierender Parameter

Beispiel:

``` cs
  void testNoSuccess() {
    var ratingSvcConn = new FakeRatingSvcConnection();
    var creditMaster = new CreditMaster();
    var validator = new CreditValidator(ratingSvcConn, creditMaster);
    var actual = validator.IsCustomerCreditWorthy(new Customer());
    Assert.That(actual, Is.False);
  }
```

---

### Fall 1: Irritierender Parameter

Beispiel:

``` cs
  void testNoSuccess() {
    var ratingSvcConn = new FakeRatingSvcConnection();
    var creditMaster = new CreditMaster();
    var validator = new CreditValidator(ratingSvcConn, creditMaster);
    var actual = validator.IsCustomerCreditWorthy(new Customer());
    Assert.That(actual, Is.False);
  }
```

# CreditMaster?

---

### Fall 1: Irritierender Parameter

Beispiel:

``` cs
  void testNoSuccess() {
    var ratingSvcConn = new FakeRatingSvcConnection();

    var validator = new CreditValidator(ratingSvcConn, null);
    var actual = validator.IsCustomerCreditWorthy(new Customer());
    Assert.That(actual, Is.False);
  }
```

# Pass Null!

---

### Fall 1: Irritierender Parameter

Beispiel:

``` cs
  void testNoSuccess() {
    var ratingSvcConn = new FakeRatingSvcConnection();

    var validator = new CreditValidator(ratingSvcConn, null);
    var actual = validator.IsCustomerCreditWorthy(new Customer());
    Assert.That(actual, Is.False);
  }
```

# Pass Null!
Parameter nach Bedarf ergänzen, wenn's knallt

---

<br />
### Fall 2:
## Verborgene Dependency
<img src='images/surprised.gif' />

---

### Fall 2: Verborgene Dependency

Beispiel:

```cs
  public class MailingListDispatcher {
    private MailService _mailService;

    public MailingListDispatcher() {
      const int clientType = 12;
      _mailService = new MailService();
      _mailService.connect();
      if(_mailService.getStatus() == MailStatus.Available) {
        _mailService.register(this, clientType);
        _mailService.setParam(clientType, MailParam.NoBounce);
      }
      ...
    }
  }
```

---

### Fall 2: Verborgene Dependency

Beispiel:

```cs
  public class MailingListDispatcher {
    private MailService _mailService;

    public MailingListDispatcher() {
      const int clientType = 12;
      _mailService = new MailService();
      _mailService.connect();
      if(_mailService.getStatus() == MailStatus.Available) {
        _mailService.register(this, clientType);
        _mailService.setParam(clientType, MailParam.NoBounce);
      }
      ...
    }
  }
```
Problem: Dependency 'MailService' schwer zugänglich

---

### Fall 2: Verborgene Dependency

Lösungsansatz
```cs
  public class MailingListDispatcher {
    private MailService _mailService;

    public MailingListDispatcher(MailService mailService) {
      const int clientType = 12;
      _mailService = mailService;
      _mailService.connect();
      if(_mailService.getStatus() == MailStatus.Available) {
        _mailService.register(this, clientType);
        _mailService.setParam(clientType, MailParam.NoBounce);
      }
      ...
    }
  }
```
Schritt 1: Parameterize Constructor

---

### Fall 2: Verborgene Dependency

Lösungsansatz
```cs
  public class MailingListDispatcher {
    private MailService _mailService;

    public MailingListDispatcher(IMailService mailService) {
      const int clientType = 12;
      _mailService = mailService;
      ...
    }
  }

  public interface IMailService {
    void connect();
    MailStatus getStatus();
    void register(MailingListDispatcher dispatcher, int type);
    void setParam(int type, MailParam);
  }
```
Schritt 2: Extract Interface

---

<br />
### Fall 3:
## Verkettete Konstruktionen
<img src='images/domino.gif' />

---

### Fall 3: Verkettete Konstruktionen

Beispiel:

```cs
  public class WaterColorPane {
    public WaterColorPane(Form border, Brush brush, Pattern backdrop) {
      ...
      var anteriorPanel = new Panel(border);
      anteriorPanel.setBorderColor(brush.getForeColor());
      var backgroundPanel = new Panel(border, backdrop);

      _cursor = new FocusWidget(brush, backgroundPanel);
      ...
    }
  }
```

---

### Fall 3: Verkettete Konstruktionen

Beispiel:

```cs
  public class WaterColorPane {
    public WaterColorPane(Form border, Brush brush, Pattern backdrop) {
      ...
      var anteriorPanel = new Panel(border);
      anteriorPanel.setBorderColor(brush.getForeColor());
      var backgroundPanel = new Panel(border, backdrop);

      _cursor = new FocusWidget(brush, backgroundPanel);
      ...
    }
  }
```

Problem:

Überwachung von "cursor" schwierig

=> Construction Blob!

---

### Fall 3: Verkettete Konstruktionen

Lösungsansatz:

```cs
  public class WaterColorPane {
    public WaterColorPane(Form border, Brush brush, Pattern backdrop) {
      ...
      var anteriorPanel = new Panel(border);
      anteriorPanel.setBorderColor(brush.getForeColor());
      var backgroundPanel = new Panel(border, backdrop);

      _cursor = new FocusWidget(brush, backgroundPanel);
      ...
    }

    void supersedeCursor(FocusWidget newCursor) {
      _cursor = newCursor;
    }
  }
```

Supersede Instance Variable

Vorsicht: Nicht in Produktionscode verwenden!

---

<br />
### Fall 4:
## Irritierende globale Dependency
<img src='images/confusion.gif' />

---

### Fall 4: Irritierende globale Dependency

Beispiel:

```cs
  public class Facility {
    public Facility(int facilityCode, string owner, PermitNotice notice) {
      var permit = PermitRepo.getInstance().findPermit(notice);
      ...
    }
  }
```

Anmerkung:

15 weiter Klassen nutzen `.findPermit()`` auf diese Weise

---

### Fall 4: Irritierende globale Dependency

Beispiel:

```cs
  public class Facility {
    public Facility(int facilityCode, string owner, PermitNotice notice) {
      var permit = PermitRepo.getInstance().findPermit(notice);
      ...
    }
  }
```

Problem:

"PermitRepository" ist als Singleton schwer zu überwachen!

---

### Fall 4: Irritierende globale Dependency

Lösungsansatz:

```cs
  public class PermitRepo {
    private static PermitRepo instance = null;

    private PermitRepo() {}

    public static PermitRepo getInstance() {
      if(instance == null) {
        instance = new PermitRepo();
      }
      return instance;
    }
    ...
  }
```

---

### Fall 4: Irritierende globale Dependency

Lösungsansatz:

```cs
  public class PermitRepo {
    private static PermitRepo instance = null;

    private PermitRepo() {}

    public static PermitRepo getInstance() {
      if(instance == null) {
        instance = new PermitRepo();
      }
      return instance;
    }

    public static void resetForTesting() {
      instance = null;
    }
    ...
  }
```

Idee 1: resetForTesting in "beforeEach" aufrufen

=> frische Instanz für jeden Test

---

### Fall 4: Irritierende globale Dependency

Lösungsansatz:

```cs
  public class PermitRepo {
    private static PermitRepo instance = null;

    private PermitRepo() {}

    public static PermitRepo getInstance() {
      if(instance == null) {
        instance = new PermitRepo();
      }
      return instance;
    }

    public static void setTestingInstance(PermitRepo newInstance) {
      instance = newInstance;
    }
    ...
  }
```
<div style='font-size:80%'>
Idee 2: Introduce Static Setter
<br />
=> Problem: Wie Instanz erstellen bei private ctor? public machen?
<br />
=> Subclass und Override mit protected ctor?
</div>

---

### Fall 4: Irritierende globale Dependency

Lösungsansatz:

```cs
  public class PermitRepo: IPermitRepo {
    private static IPermitRepo instance = null;

    private PermitRepo() {}

    public static IPermitRepo getInstance() {
      if(instance == null) {
        instance = new PermitRepo();
      }
      return instance;
    }

    public static void setTestingInstance(IPermitRepo newInstance) {
      instance = newInstance;
    }
    ...
  }

  public interface IPermitRepo {
    Permit findPermit(PermitNotice notice);
  }
```

Idee 2b: + Extract Interface

---

### Fall 4: Irritierende globale Dependency

<br />
<br />
## Idee 3:

Globale Variablen/Dependencies eliminieren!

---

<br />
### Fall 5:
## Schreckliche Include-Dependency
<img src='images/kingkong.gif' />

---
