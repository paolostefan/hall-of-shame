# 👹 NoDoList: bestialità da evitare in PHP

---

Many thanks to an **Anonymous Historical developer**, hereafter "Damn Developer" for the ~~coding 💩s~~ invaluable inspiration.

---

- [👹 NoDoList: bestialità da evitare in PHP](#-nodolist-bestialità-da-evitare-in-php)
  - [Inserire credenziali nel codice](#inserire-credenziali-nel-codice)
  - [Scrivere i log usando file_put_contents()](#scrivere-i-log-usando-file_put_contents)
    - [Come ottenere lo stesso risultato](#come-ottenere-lo-stesso-risultato)
  - [`new stdClass()` è già illegale in molti Stati](#new-stdclass-è-già-illegale-in-molti-stati)
  - [Codice inutile, parte 1](#codice-inutile-parte-1)
  - [Inserire logiche opache, instabili, peggio se nel posto sbagliato](#inserire-logiche-opache-instabili-peggio-se-nel-posto-sbagliato)
    - [Bonus Damn Developer 👹👹👹](#bonus-damn-developer-)
    - [Altro esempio: un setter che è un concatenatore](#altro-esempio-un-setter-che-è-un-concatenatore)
    - [Lo stesso effetto lo otterremmo così](#lo-stesso-effetto-lo-otterremmo-così)
  - [Mischiare _business logic_ e _presentation layer_](#mischiare-business-logic-e-presentation-layer)
  - [Switch inutili](#switch-inutili)
  - [Scrivere codice che non dovrebbe ma funziona](#scrivere-codice-che-non-dovrebbe-ma-funziona)
  - [Scrivere codice che non dovrebbe ma funziona / 2](#scrivere-codice-che-non-dovrebbe-ma-funziona--2)
    - [Risultato](#risultato)
    - [👹 👹 👹 Doppio bonus Damn Developer](#---doppio-bonus-damn-developer)
  - [Usare eval()](#usare-eval)
    - [Soluzione alternativa](#soluzione-alternativa)
  - [Inserire commenti che non spiegano niente](#inserire-commenti-che-non-spiegano-niente)
    - [👹 Bonus Damn Developer](#-bonus-damn-developer)
  - [Usare @author](#usare-author)
  - [Lanciare eccezioni con messaggi inutili](#lanciare-eccezioni-con-messaggi-inutili)
  - [Catturare Exception per lanciarne altre](#catturare-exception-per-lanciarne-altre)
    - [Come evitare questa bestialità](#come-evitare-questa-bestialità)
      - [Soluzione 1: **togliete** il try-catch](#soluzione-1-togliete-il-try-catch)
      - [Soluzione 2: evitate di lanciare Exception nel blocco `catch`](#soluzione-2-evitate-di-lanciare-exception-nel-blocco-catch)
      - [Soluzione 3: rilanciate la **stessa** Exception](#soluzione-3-rilanciate-la-stessa-exception)
  - [Codice per esseri umani, Part I: non riassegnate le variabili](#codice-per-esseri-umani-part-i-non-riassegnate-le-variabili)
  - [Codice per esseri umani, Part II: date nomi sensati alle variabili](#codice-per-esseri-umani-part-ii-date-nomi-sensati-alle-variabili)
  - [json_encode() trasforma array associativi in oggetti](#json_encode-trasforma-array-associativi-in-oggetti)
  - [Estendere stdClass](#estendere-stdclass)
  - [Scrivere istruzioni dopo throw](#scrivere-istruzioni-dopo-throw)
  - [Far restituire un valore al costruttore](#far-restituire-un-valore-al-costruttore)
  - [Mantenere codice che non fa nulla](#mantenere-codice-che-non-fa-nulla)
  - [Scrivere logiche ad hoc per un solo utente](#scrivere-logiche-ad-hoc-per-un-solo-utente)
  - [Lasciare markup inutile in un commento HTML](#lasciare-markup-inutile-in-un-commento-html)
    - [Soluzione 1](#soluzione-1)
    - [Soluzione 2](#soluzione-2)

## Inserire credenziali nel codice

Partiamo da una considerazione semplice: non è il caso di salvare password e altri dati sensibili in un repo Git che probabilmente finirà in server che NON controlliamo: non abbiamo modo di sapere **chi** accederà a quelle password e **quale uso ne farà**.

Ovvio? Direi di sì.

Però io continuo a trovare password nei repository Git, parcheggiate lì perché ... c'era fretta? Era più comodo così? Nessuno aveva pensato che il repo venisse visto da occhi indiscreti? Non sapevamo come fare altrimenti?

Tutti motivi non validi.

## Scrivere i log usando file_put_contents()

**Don't try this at home:**

```php
file_put_contents("/var/log/ilmiofiledilog.log", "Messaggio di log\n", FILE_APPEND);
```

### Come ottenere lo stesso risultato

La funzione `error_log()` scrive i messaggi di log nel file configurato in **php.ini** alla voce omonima. Per scrivere messaggi in un altro file, non avete che da chiederlo a PHP chiamando `ini_set("error_log", $percorsoDelFileDiLog)`.

```php
ini_set("error_log", "/var/log/pippo.log");
error_log("Messaggio di log"); // verrà scritto in /var/log/pippo.log
```

## `new stdClass()` è già illegale in molti Stati

- Vi serve un `object` a tutti i costi?
- Odiate gli array associativi?
- Soffrite di cefalee croniche?
- Siete insoddisfatti della vostra vita?
- Dormite male?

Usare `new \stdClass()` **non** risolverà nessuno di questi problemi. Non usatelo, specie per le cefalee.

Per la questione degli `object`, però, potete usare un _type cast_ a _object_, come segue:

```php
$ilMioAmatoOggetto = (object)[
    'campoNumerico' => 123,
    'campoStringa' => 'lorem ipsum',
    'campoArray' => (object)[
        'campoNumerico' => 123,
        'campoStringa' => 'lorem ipsum',
    ],
];
```

## Codice inutile, parte 1

In una classe derivata ci sono tutti i metodi della classe _madre_. So che voi lo sapete, perché siete programmatori degni di questo nome, ma chi ha scritto questo codice no (nel senso che non era un programmatore):

```php
    public function __call($method, $argv)
    {
        return parent::__call($method, $argv);
    }
```

## Inserire logiche opache, instabili, peggio se nel posto sbagliato

Chiarisco subito i due termini _opaca_ e _instabile_.

1. Per _logica opaca_ intendo funzioni e metodi dal comportamento **inatteso** o **non intuibile** quando si legge il codice chiamante.

   **Esempio**: una funzione di nome `cancellaUtente($utente)` che **non** cancella l'utente `$utente` ma ne _imposta il nome_ a `"StatoZero"`; un metodo di nome `getName()` che restituisce il campo privato `$this->name`, ma oltre a questo (atteso), imposta anche altri campi e salva il record a database.

2. Per _logica instabile_ intendo codice che può introdurre **effetti collaterali** inattesi e non documentati, legati al suo input.

   **Esempio**: una funzione `creaFile($config)` che crea un file (atteso), ma oltre a ciò, se `$config['zibibbo']` vale esattamente `12345`, _cancella tutti i file creati finora_, dopo averli _zippati_ e inviati via email a un indirizzo generato casualmente.

Fino a qui, gli esempi in questo paragrafo erano prodotti della mia fantasia.

Veniamo alla dura realtà: codice trovato ~nel solito repository~ in un repository.

```php
class UnaExceptionConLogicheOpache extends \InvalidArgumentException
{
    public function __construct($o)
    {
        parent::__construct('Ciccio\Exception');
        if (!empty($o->is_email)) {
            $this->send($o->title, $o->msg);
        }
        if (isset ($o->file) && is_file($o->file)) {
            unlink ($o->file);
        }
    }

    private function send($title, $msg){
        // invia un messaggio email!
    }
}
```

Un costruttore dovrebbe inizializzare campi, non **inviare messaggi email** o **cancellare** file... 😫 ma è solo un mio parere.

### Bonus Damn Developer 👹👹👹

Questa bestialità, oltre ad avere logiche opache e instabili, è più insidiosa perché si trova nel **costruttore** di una Exception... quindi basterà istanziarla per rischiare di cancellare file o inviare email.

### Altro esempio: un setter che è un concatenatore

Un altro esempio che viene dal mondo reale.

```php
class MaPercheFareQuesto {
    public function __set ( $s , $v ) {
        $this->$s .= $v;
 }
}
```

Non riesco a capire la necessità di un magic method che, quando imposti un campo della classe, anziché impostarlo lo concatena. Problema mio.

Il problema di chi verrà dopo di questa bestia, invece, sarà che una serie di chiamate di questo tipo non produrrà quello che chiunque si aspetterebbe.

```php
$a = new MaPercheFareQuesto();
$a->message = 'Ciao a tutti';
$a->message = 'Errore 201';
$a->message = 'Nessun errore';

echo $a->message; // Output: Ciao a tuttiErrore 201Nessun errore
```

### Lo stesso effetto lo otterremmo così

L'operatore di concatenazione esiste in PHP dalla notte dei tempi. In questo caso basterebbe usare questo codice:

```php
$a = (object)['message'=>''];
$a->message = 'Ciao a tutti';
$a->message .= 'Errore 201';
$a->message .= 'Nessun errore';
```

## Mischiare _business logic_ e _presentation layer_

Ok, anche Wordpress nei suoi template mischia markup HTML e tag `<?php ?>` a non finire. Ma:

1. questo non la rende una _pratica ottima_;
2. se uno script è incaricato di renderizzare una pagina HTML, e invece nel mezzo fa _altre cose_, prima o poi vi ritroverete a chiedervi: "Per quale motivo sta succedendo **{una di queste altre cose}**?"

## Switch inutili

In una codebase (una a caso) ho trovato **decine** di spezzoni come questo (non esagero):

```php
switch ($variabile) {
    default:
        $this->faiQualcosa();
}
```

Un costrutto `switch` ha senso di esistere solo se ha **più di due** `case`.

Se ha solo _default_ è ovviamente superfluo e può essere eliminato senza modificare nient'altro.

Se ha **uno o due** `case` (escluso `default`), lo switch può diventare più leggibile sotto forma di **if/elseif/else**.

Nell' esempio seguente lo switch è sempre inutile, ma sostituirlo richiede un minimo di sforzo in più:

```php
switch ($month) {
    case '01':
        if ($day == '25') {
            $foo = true;
        }
        break;
    default:
        if ($day == '05') {
            $foo = true;
        }
        break;
}
```

Equivale a:

```php
if ($month === '01') {
    if ($day == '25') {
        $foo = true;
    }
} else {
    if ($day == '05') {
        $foo = true;
    }
}
```

Questo pezzo di codice può essere accorciato ulteriormente, ma **solo se** `$foo` deve valere `false` in tutti gli altri casi.

```php
$foo = $month === '01'? ($day == '25') : ($day == '05');
```

Da 12 linee a una sola. 👍

Ovvio che **se vi pagano a righe**, accorciare il codice non conviene.

## Scrivere codice che non dovrebbe ma funziona

Il codice qui sotto è concettualmente errato: persiste (salva) solo **una** Entity ogni **N**. L'istruzione `persist()` andrebbe spostata prima e fuori dall'if (come pure il controllo degli errori, ma è un dettaglio...).

_Attenzione anche agli utilissimi commenti, mantenuti dall'originale._

```php
for ($i = 0; $i < $total_records; $i++) {
    # altre cose...

    // START flush

    if (!($i % $this->flush_every) && !$this->dryrun && !count($this->errors)) {
        $this->_em->persist($entity);
        $this->_em->flush(); // execute flush
        $this->_em->clear(); // clear
    }
    // END flush
}
```

Eppure, **questa bestialità funziona**. Perché? Tenetevi...

---

😱 Funziona **solo** perché `$this->flush_every` è uguale a `1`. 😱

---

## Scrivere codice che non dovrebbe ma funziona / 2

Questa bestialità è più sottile, se di sottigliezze si può parlare nella _Gora dell'eterno fetore_...

Consideriamo la seguente classe (ho mantenuto gli utilissimi commenti originali):

```php
class NonScriveteClassiComeQuesta {

    private $pippo;

    /**
     * __call
     */
    public function __call($method, $argv)
    {
        // set
        if(strpos($method, "set") === 0) {
            $prop = substr($method, 3);
            $this->$prop = $this->castValue($argv[0],
             isset($argv[1]) ? $argv[1] : "string");
            return true;
        }

        // get
        if(strpos($method, "get") === 0) {
            $prop = substr($method, 3);
            return $this->$prop;
        }
        return false;
    }
}
```

In questa classe l'autore definisce un _magic method_ che (come l'asso piglia tutto della briscola) viene invocato quando in un'istanza di questa classe viene chiamato un metodo non definito.

Nelle intenzioni dello sciagurato autore, questo metodo risparmierebbe la fatica di scrivere due metodi (_setter_ e _getter_) per ciascun campo privato della classe, ma in realtà fa più male che bene.
Consideriamo questo codice chiamante:

```php
$bad = new NonScriveteClassiComeQuesta();
$bad->setCode('12345');
```

Cosa dovrebbe succedere?

Se avessimo un setter (e magari anche un campo privato di nome `$Code`, che non c'è come non c'era nel codice originale), al ritorno di questa chiamata ci aspetteremmo che il campo `$bad->Code` valesse `'12345'`.

Invece ecco ciò che succede:

1. PHP controlla se `setCode()` è un metodo di questa classe;
2. siccome la classe non è derivata, si ferma qui e non ripercorre gli antenati della classe (non ce ne sono);
3. il metodo `setCode()` **non esiste**, quindi PHP controlla se esiste il metodo `__call()`;
4. `__call()` c'è, PHP lo chiama passandogli come _primo_ argomento il nome del metodo inesistente (la stringa `"setCode"`), e come _secondo_ argomento l'array degli argomenti, cioè `['12345']`.

In altre parole PHP traduce dietro le quinte `$bad->setCode('12345');` in `$bad->__call('setCode',['12345']);`

Dentro `__call()` verrà eseguito questo spezzone:

```php
if(strpos($method, "set") === 0) {
    $prop = substr($method, 3);  // $prop = 'Code'
    $this->$prop = $this->castValue($argv[0], isset($argv[1]) ? $argv[1] : "string");
    return true;
}
```

Qui varchiamo la soglia dell'antro della Bestia.

Se usiamo un IDE tipo PHPStorm che evidenzia i metodi indefiniti (o se siamo svegli), leggendo il corpo dell'`if` qui sopra la prima idea che ci può venire è: `castValue()` non è definito, causerà un'Exception!

---

😱 **Invece no**. Il codice bestiale -anche se non fa quello che ci aspettiamo - **non** genera Exception. Questa chiamata a `castValue()` genera **ricorsivamente un'altra chiamata** a `__call()`, e siccome il nome del metodo (`castValue`) non inizia né per **set** né per **get**, `__call()` non entra in nessuno dei due `if()`... e restituisce `false`!

---

### Risultato

```php
$bad->setCode('12345'); // $bad->Code vale... '12345'? No, vale false!
```

Per non fare la fatica di scrivere un _setter_, il codice:

- imposta anche **campi non definiti** nella classe (cattiva pratica);
- va più lento del dovuto (le chiamate a `__call` sono un giro in più, la definizione dinamica di campi fa rallentare); ma soprattutto...
- imposta _qualsiasi_ campo a `false` senza darci un indizio di quel che sta accadendo! A conferma del fatto che le Exception non sono il male. Una manifestazione del male è scrivere codice come questo.

---

### 👹 👹 👹 Doppio bonus Damn Developer

In questa classe infernale:

1. è lecito (cioè, non genera Exception) chiamare setter e getter arbitrari, con un numero arbitrario di argomenti! 👹

2. Siccome gli `if()` non controllano la lunghezza del nome del metodo, è lecito chiamare il metodo inesistente `set()` che imposterà un attributo **senza nome** (`$this->''`) nell'istanza della classe, e per quello che abbiamo visto lo imposterà a `false`. _Chapeau_ 🎩 👹👹

## Usare eval()

Ogni volta che usate `eval()`, un gattino viene ucciso. 🐈🔪

Praticamente ogni utilizzo della funzione `eval()` ha un'alternativa più leggibile, più facile da mantenere, e meno contorta. Gli IDE non riconoscono i costrutti dentro `eval()`. Se il codice eseguito da `eval()` imposta variabili, carica definizioni di classi, o include altri file, sia l'IDE che il disgraziato sviluppatore che legge il codice avranno problemi a capire cosa succede; per non parlare poi della inutile difficoltà di testare un codice che fa `eval()`.

Ma se usate `eval()` per fare quel che segue, state rasentando sia l'idiozia che il reato penale:

```php
$aLines = file ("some.class.php"); // leggi un file php
array_shift ($aLines); // togli la prima riga (il tag di apertura!!!)
// Fai eval del file anteponendo un namespace (???)
eval("namespace Foo;\n" . implode("\n", $aLines));
```

### Soluzione alternativa

Salva il contenuto del file (inserendo `namespace Foo;`) in un altro file.

## Inserire commenti che non spiegano niente

...Ovvio, no? Ho trovato (grazie Damn Developer) commenti come quelli qui sotto.

```php
/**
 * add
 */
public function add (\stdClass $oReq)
{
} // add

/**
 * update
 */
public function update (\stdClass $oReq)
{
} // update

/**
 * cancel
 */
public function cancel (\stdClass $oReq)
{
} // cancel

/**
 * restore
 */
public function restore (\stdClass $oReq)
{
} // restore
```

### 👹 Bonus Damn Developer

Nel sorgente originale, i metodi erano _tutti vuoti_ come qui sopra.

## Usare @author

A meno che non stiate sviluppando software open source _e_ lo stiate scrivendo **bene**, lasciate perdere le annotazioni `@author`. Specie se avete scritto bestialità come quelle riportate in questo post.

Se chi viene dopo di voi non sa chi siete, è meno probabile che venga a cercarvi a casa per rompervi le rotule. Damn Developer, che inseriva miriadi di `@author` a destra e a manca in sorgenti colmi di bestialità, ha evitato l'ortopedia solo perché ormai è fuggito all'estero.

## Lanciare eccezioni con messaggi inutili

👹 Bestialità n. 520:

```php
throw new \Exception("ERROR");
```

Un errore con messaggio "ERROR", oltre che un pleonasmo, è inutile. Una buona pratica sarebbe:

1. lanciare una _Exception_ specifica (una sottoclasse di `\Exception`, eventualmente definita nella nostra applicazione);
2. passare al costruttore dell'_Exception_ un messaggio significativo su **cosa** è andato storto, o cosa si aspettava e non ha trovato. Insomma: ditemi qualcosa, cacchio!

## Catturare Exception per lanciarne altre

👹 Bestialità n. 1234:

```php
try {
    // ...
} catch (\Exception $e) {
    throw new Exception (str_replace("\n", "<br>", $e->__toString()));
}
```

La funzione magistrale delle Exception è fornire informazioni non solo sul **tipo** di errore che si è verificato, ma anche sul **dove** (nel codice) si è verificato l'errore. Un codice come quello dell'esempio sopra rende **doppiamente** inutile l'Exception:

1. fa perdere l'informazione sul tipo, perché rilancia un'eccezione generica;
2. fa perdere l'informazione sul posto in cui è avvenuta l'eccezione (e sullo stack trace), perché ne lancia una nuova all'interno del `catch`: quindi per PHP sarà come se l'errore si fosse verificato nel blocco `catch`.
3. 👹 👹 👹 **Bonus Damn Developer**: oltre ai punti 1 e 2, il codice qui sopra rende inutile l'eccezione **all'unico scopo di rimpiazzare i newline con i `<br>`**.

### Come evitare questa bestialità

#### Soluzione 1: **togliete** il try-catch

A meno che non faccia operazioni necessarie e **rilevanti**, un blocco try-catch **può essere tolto**.

#### Soluzione 2: evitate di lanciare Exception nel blocco `catch`

Se il blocco `catch` fa operazioni _necessarie_ ma il codice **non si deve per forza fermare**, evitate di lanciare un'altra eccezione nel blocco `catch`.

#### Soluzione 3: rilanciate la **stessa** Exception

Se il `catch` fa operazioni _necessarie_ e il codice **si deve fermare**, rilanciate **la stessa istanza** dell'eccezione, così:

```php
try {
    // ...
} catch (\Exception $e) {
    // fai le cose irrinunciabili
    // ...
    throw $e;
}
```

## Codice per esseri umani, Part I: non riassegnate le variabili

Sempre grazie a Damn Developer:

```php
$a = $q->getArrayResult ();
$s1 = $a[0]["campo_1"];
$s2 = $a[0]["campo_2"];
$s3 = $a[0]["campo_3"];
$a = $a[0][0];
$a["campo_1"] = $s1;
$a["campo_2"] = $s2;
$a["campo_3"] = $s3;
```

## Codice per esseri umani, Part II: date nomi sensati alle variabili

Ecco un altro spezzone di codice realmente esistito:

```php
$a = $this->getRecords (); // $a ?
$oRes = new \stdClass;
$oRes->records = array ();
$oRes->codes = array ();
$totalND = 0;
$countND = 0;
foreach ($a as $o) { // $o ???
    if ($o->code != "") {
        $oRes->codes [] =$o->code;
        $oRes->records [] = array (
            "name" => $o->code,
            "num" => $o->num,
            "y" => (float) $o->total,
            "position" => $o->position,
            "description" => $o->description
        );
    } else {
        $totalND += (float) $o->total;
        $countND += $o->num;
    }
}
$oRes->records [] = array ("name" => "NV", "num" => $countND, "y" => (float) $totalND, "description" => "non valutato", "uri" => $o->uri) ;
$oRes->codes [] = "NV";
return $oRes;
```

## json_encode() trasforma array associativi in oggetti

Se date in pasto un array associativo a `json_encode()`, quest'ultimo lo trasformerà in un _oggetto_: **non serve** l'ennesimo _type cast_ a _object_.

```php
json_encode((object)['pippo'=>123, 'pluto'=>'Ciao']);
json_encode(['pippo'=>123, 'pluto'=>'Ciao']);
```

## Estendere stdClass

**Tutte** le classi estendono `\stdClass`, di default: non serve dirlo a PHP.

🛑 Non scrivete questo:

```php
class UnaClasseQualsiasi extends \stdClass
{
}
```

ma questo:

```php
class UnaClasseQualsiasi
{
}
```

## Scrivere istruzioni dopo throw

Una _exception_ causa l'uscita dallo script, oppure viene gestita da un blocco `catch`: in ogni caso, tutte le istruzioni **dopo** `throw` non verranno eseguite.

Non ha senso scrivere:

```php
throw new Exception ('Messaggio di errore');
// tutto quello che viene dopo throw non verrà eseguito
echo "Stiamo per uscire..."
exit 1;
```

## Far restituire un valore al costruttore

Il costruttore di una classe restituisce l'istanza della classe, sempre e invariabilmente, e senza bisogno di istruzioni `return`. Le istruzioni `return` verranno ignorate.

```php
class MancanzaDiClasse {

    private $pippo;

    public function __construct ()
    {
        try {
            // ...
        }
        catch (\Exception $e) {
        }

        return  $this->pippo; // Non farà quello che pensiamo.
    }
}
```

## Mantenere codice che non fa nulla

Quando in una codebase uno sviluppatore trova un metodo con questo contenuto, è suo dovere morale **perlomeno** rimuoverne il contenuto.

```php
    public function nonFaccioNiente($argomento)
    {
        try {

        } catch (\Exception $e) {

        }
    }
```

Una mossa più audace sarebbe **rimuovere in toto il metodo** per vedere cosa succede.

## Scrivere logiche ad hoc per un solo utente

Bestialità cugina della precedente: riguardano entrambe l'_hardcoding_ in script PHP di valori che dovrebbero essere salvati a database.

```php
if ($user->getLogin() === 'ciccio@example.com') {
    $variabile = "Valore dedicato a Ciccio";
}
```

Lo stesso comportamento si può ottenere in modi umani, ad esempio -nel caso qui sopra- usando un flag e i campi dinamici dell'oggetto `$user`, anziché scrivere un indirizzo email nel codice.

```php
if ($user->getDedicatedFlag()) {
    $variabile = "Valore dedicato a ".$user->getFirstName();
}
```

## Lasciare markup inutile in un commento HTML

I commenti HTML, molto spesso, non servono al difuori dell'ambiente di sviluppo: aumentano la dimensione del contenuto scaricato dal browser dell'utente, ma soprattutto possono fornire informazioni compromettenti a utenti malintenzionati.

👹 Bestialità n. 8920:

```php
<!--
<style type="text/css">
    <?php CSS::compress("style.css"); ?>
</style>
-->
```

Questo codice è doppiamente 💩 perché:

1. mantiene un contenuto inutile nel markup;

2. peggio, occupa inutilmente la CPU del server, attivando l'interprete PHP per _comprimere_ e includere un file CSS che **non verrà utilizzato**! Per queste finezze ringraziamo sempre il <strike>deplorevole</strike> buon Damn Developer.

### Soluzione 1

Cancellate il codice che avete commentato. Tanto in Git potete ripristinarlo.

Se vi piaceva tanto da non poterne fare a meno, spostatelo in un altro file (inutilizzato), e se volete, menzionate questo spostamento in un commento _PHP_ nel file originale, così perlomeno non verrà stampato nel markup. Sì, so che poche righe fa ho detto di non attivare l'interprete PHP per niente, ma se volete mantenere codice inutile, questa soluzione è più sicura.

```php
<?php
// Qui c'era una bestialità, spostata ora in ../shit/12345.txt
?>
```

### Soluzione 2

Usate un compressore/builder/_minifier_ per il codice HTML.

Non è agevole usare un _minifier_ se intervallate HTML e PHP nello stesso file sorgente: questa è una delle soluzioni che vi costringeranno a organizzare meglio il vostro codice.
