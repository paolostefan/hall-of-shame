# 👹 NoDoList: bestialità da evitare in Javascript

## Usare setInterval() senza criterio

Codice come questo occupa inutilmente l'interprete JS del browser: ogni 20 millisecondi esegue una sbrodolata di codice, la maggior parte delle volte inutile.

L'aspetto peggiore su architetture multicore è che, pur avendo una cpu al 100%, non te ne accorgi (una CPU al 100% su 8 fa circa il 12% di impegno sul totale).

```js
// START scrollbars
idInterval = window.setInterval (function () {
  $ (".scrollable:visible").each ( function () {
    if ($ (this).attr ("h")) {
        if (parseInt ($(this).attr ("h")) > 0) {
            $ (this).height ($ (this).attr ("h"));
        } else {
            iH = $(window).height () - ($ (this).offset ().top + $ ("#div-wrapper").scrollTop ()) + parseInt ($ (this).attr ("h"));
            if (iH < $ (this).find (".jspPane").height () && iH > 0) {
                $ (this).height (iH);
            }
            else {
                $ (this).height ($ (this).find (".jspPane").height ());
            }
        }
    }
    else {
        iH = $(window).height () - ($ (this).offset ().top + $ ("#div-wrapper").scrollTop ()) - 60;
        if (iH < $ (this).find (".jspPane").height () && iH > 0) {
            $ (this).height (iH);
        }
        else {
            $ (this).height ($ (this).find (".jspPane").height ());
        }
    }
    $ (this).jScrollPane ({
        mouseWheelSpeed : 30,
    });
    // update header;
    if ($ (this).attr ("header") !== "") {
        if ($ (this).attr ("header")) {
            var oHeader = $ ("." + $ (this).attr ("header"));
        } else {
            var oHeader =  $ (this).prev ("thead");
        }
        oHeader.children ('tr').wrapAll( "<div class='jspContainerHeader'></div>" ).wrapAll( "<div class='jspPaneHeader'></div>" );
        oHeader.find (".jspContainerHeader")
                .width ($ (this).find (".jspContainer").width())
                .css ("background-color", oHeader.find ("th").eq(0).css ("background-color"));
        oHeader.find (".jspPaneHeader").width ($ (this).find (".jspPane").width())
    }
  });
},20);
```

## Sfruttare comportamenti poco stabili

Nel codice che segue:

- str è una stringa
- str.match() restituisce un array di stringhe
- l'intenzione dell'if è verificare se str contiene un elenco di interi separati da virgola

```js
if (str != str.match(/(\d+)/g)) {
    // secondo l'autore, arrivare qui è indice di errore
}
```

In un linguaggio fortemente tipizzato, confrontare queste due entità non avrebbe senso; infatti, se in JS vengono confrontate con '!==', il test darà sempre esito positivo in quanto sono di due tipi diversi.

Ma...
se effettuiamo il confronto con `!=`, JS convertirà dietro le quinte il secondo membro in *stringa*, e se `str` è composto solo da numeri e virgole (NOTA: **senza alcuno spazio**!), la *rappresentazione* in stringa dell'array di match sarà (per una coincidenza) esattamente identica a `str`.

Semplicemente aberrante.
