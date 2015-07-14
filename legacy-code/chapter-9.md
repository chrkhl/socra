# Hilfe
### ich kann diese Klasse nicht testen!
<img src='images/help.gif' />
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
