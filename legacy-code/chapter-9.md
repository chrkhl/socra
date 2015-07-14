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

Schritt 1: Extract Interface

``` cs
  public interface IRatingSvcConnection {
    public void Connect() {...}
    public void Disconnect() {...}
    public void Retry() {...}
    public RatingReport ReportFor(int userId) {...}
  }
```

---

### Fall 1: Irritierender Parameter
Lösungsansatz

Schritt 2: Fake implementieren

``` cs
  public FakeRatingServiceConnection: IRatingSvcConnection {
    public void Connect() { }
    public void Disconnect() { }
    public void Retry() { }
    public RatingReport ReportFor(int userId) { return null; }
  }
```

---

### Fall 1: Irritierender Parameter
Lösungsansatz

Schritt 3: Fake im Test verwenden

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
