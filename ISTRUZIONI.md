# Scansiona e invia

Inquadri un documento e il mirino ne segna i bordi in giallo mentre lo muovi; allo scatto l'app lo ritaglia, lo raddrizza, lo pulisce e lo mette in un PDF. Poi un pulsante lo manda per mail, un altro su WhatsApp. Niente passaggio da Drive, niente archivio da svuotare.

Una precisazione su Drive: il suo scanner non è riutilizzabile da fuori. È un componente nativo di Android (ML Kit Document Scanner) che solo le app *installate* possono richiamare, e quella strada l'abbiamo esclusa per via dell'MDM. Il riconoscimento dei bordi qui dentro è quindi scritto da zero e gira tutto nel telefono.

Funziona guardando i contorni, non la luminosità: su ogni fotogramma si calcola il gradiente con un operatore di Sobel, si tengono solo i punti che separano davvero due materiali (il salto di luminosità misurato cinque pixel al di qua e al di là del bordo è grande sul confine carta-tavolo e quasi nullo su una riga di testo, che ha carta da entrambe le parti), e su quei punti una trasformata di Hough guidata dal gradiente cerca le quattro rette dominanti, a due coppie parallele, che intersecate danno i vertici. Questo rende il riquadro insensibile ai gradienti di luce: un tavolo illuminato di sbieco non viene più confuso con la carta. Se i bordi sono troppo poco contrastati si ripiega sul vecchio metodo della macchia chiara, con un controllo che scarta le macchie sbordate. Il riquadro è poi mediato fra fotogrammi, così non tremola, ma si aggancia subito quando muovi il telefono.

Le proporzioni del foglio non si deducono dalla lunghezza dei lati nella foto, che per un foglio ripreso di sbieco sono sbagliate, ma si ricavano dalla proiezione col metodo di Zhang e He: imponendo che i lati del rettangolo siano perpendicolari si stima la focale dell'obiettivo, e da quella il vero rapporto fra larghezza e altezza. Un A4 esce con le proporzioni di un A4 anche se l'hai fotografato da trenta gradi di lato.

È una web app: si apre nel browser del telefono e si aggiunge alla schermata Home come un'icona qualsiasi. **Non installi nessuna applicazione**, quindi le regole dell'MDM aziendale sulle sorgenti sconosciute non entrano in gioco. Tutto il lavoro (ritaglio, raddrizzamento, creazione del PDF) avviene dentro il telefono: nessuna foto viene caricata su un server, perché un server non c'è.

## I file

`index.html` è l'applicazione completa, codice e grafica inclusi. Da sola basta a far funzionare tutto.

`manifest.webmanifest`, `sw.js` e `icon.svg` servono solo a due rifiniture: l'icona sulla schermata Home e il funzionamento anche senza rete. Se li tieni accanto a `index.html` li usa, se mancano l'app funziona comunque.

`_verifica.html`, `_verifica_mirino.html` e `_verifica_geometria.html` sono i banchi di prova con cui ho controllato rilevamento bordi, raddrizzamento, filtri, struttura del PDF, catena di scale del mirino, proporzioni ricavate dalla prospettiva e tenuta del riconoscimento su una scena con luce sbilanciata. Aprili nel browser e si eseguono da soli, riga per riga. Servono solo a me: puoi cancellarli.

## Mettere l'app online

Serve un indirizzo `https://`. Non è una formalità: senza di esso Android blocca l'accesso alla fotocamera e la condivisione dei file, e l'app non può funzionare. Aprire il file con un doppio clic dal telefono non basta.

### Strada consigliata: GitHub Pages

Gratuita, permanente, e funziona anche da un PC gestito dall'azienda perché passa tutto dal browser.

1. Crea un account su github.com, se non ne hai già uno.
2. Clicca su **New repository**, dai un nome qualsiasi (per esempio `scanner`), lascia **Public** e crea.
3. Nella pagina del repository vuoto clicca **uploading an existing file** e trascina dentro i quattro file: `index.html`, `manifest.webmanifest`, `sw.js`, `icon.svg`. Poi **Commit changes**.
4. Vai su **Settings**, voce **Pages** nel menù a sinistra. Sotto *Branch* scegli `main`, cartella `/ (root)`, e premi **Save**.
5. Aspetta un paio di minuti e ricarica: in cima compare l'indirizzo, del tipo `https://tuonome.github.io/scanner/`.

Quell'indirizzo è la tua app. Mandatelo su WhatsApp da solo, così lo apri comodamente dal telefono.

### Strada più rapida: Netlify Drop

Se vuoi provarla subito: vai su `app.netlify.com/drop`, trascina la cartella con i quattro file nel riquadro e in una decina di secondi ottieni un indirizzo `https://` funzionante. Senza registrarti il sito resta attivo per poche ore, con un account gratuito resta per sempre.

### Se l'azienda ha già uno spazio web

Qualsiasi cartella raggiungibile in https va bene, anche sulla intranet: copia i quattro file e apri l'indirizzo. Non serve nessun software lato server.

## Metterla sulla schermata Home

1. Apri l'indirizzo con **Chrome** sul telefono (non con il browser interno di WhatsApp o della posta: apri il link e scegli "Apri in Chrome").
2. Tocca i tre puntini in alto a destra.
3. Scegli **Installa app** se compare, altrimenti **Aggiungi a schermata Home**.

Da quel momento l'icona si comporta come un'app normale: si apre a schermo pieno, senza barra degli indirizzi, e funziona anche senza rete.

## Prima configurazione

Tocca l'ingranaggio in alto a destra e compila: nome del destinatario, indirizzo mail, numero WhatsApp con il prefisso internazionale (`+39…`), e quale dei due canali usi più spesso. Il canale scelto diventa il pulsante giallo in prima posizione. Si fa una volta sola: le impostazioni restano nel telefono.

Sono utili anche l'oggetto della mail e l'inizio del nome file: data e ora vengono aggiunte da sole, così niente si sovrascrive.

La voce **qualità** decide quanti pixel ha la pagina finita: Leggera 1700, Equilibrata 2250, Alta 2900 sul lato lungo, cioè circa 145, 190 e 245 punti per pollice su un A4. Alta è la scelta giusta per documenti con timbri, firme o caratteri piccoli; scendi a Leggera se la posta aziendale rifiuta allegati grossi. Tieni presente che il limite vero spesso non è questo ma la risoluzione della foto: un foglio che occupa metà dell'inquadratura dà una pagina più piccola di quanto il livello consentirebbe. Per questo l'app chiede al telefono il massimo che concede al video e, quando il browser lo permette, fa un vero scatto fotografico a piena risoluzione invece di prendere il fotogramma dell'anteprima.

## Come si usa

Sfiora lo schermo, in qualsiasi punto: si apre il mirino. Appoggia il documento su un piano, anche di sbieco, e inquadralo: quando il contorno giallo circonda il foglio la scritta in basso diventa "Foglio rilevato". Scatta con il pulsante giallo. La pagina viene ritagliata e raddrizzata subito, e il mirino resta aperto per il foglio successivo: scatti tutte le pagine di fila e poi tocchi **Fatto**. Se il telefono lo consente compare anche **Luce**, utile sui documenti in penombra.

Se il ritaglio automatico non ti convince, tocca la pagina e poi **Modifica**: trascini i quattro angoli, ruoti, scegli il filtro. **Migliora** sbianca la carta e toglie il giallo delle luci al chiuso, **Bianco e nero** è quello giusto per i fogli di solo testo da stampare o archiviare.

Poi tocchi **Mail** o **WhatsApp** e si apre il pannello di condivisione di Android con il PDF già allegato.

## Scontrini

Gli scontrini sono il caso più difficile, perché sono stretti e lunghi: diverse soglie geometriche pensate per un foglio A4 li escludevano per costruzione, e le ho allargate. La più importante riguardava la distanza minima fra due lati opposti, che per i due bordi lunghi di uno scontrino è la sua larghezza: era fissata al 20% del lato corto dell'inquadratura, e uno scontrino ci stava sotto. Il mirino esamina anche fotogrammi più grandi, 320 pixel invece di 240, perché su un oggetto così sottile i pixel contano.

Due accortezze quando ne scansioni. Tienilo tutto dentro l'inquadratura, estremi compresi: se un angolo esce, l'app preferisce non proporre un ritaglio piuttosto che proporne uno sbagliato, e per uno scontrino molto lungo conviene allora scattare da più lontano o usare **Tutta la foto** e ritagliare a mano. E appoggialo su qualcosa di colore diverso dalla carta: su un tavolo bianco il bordo non si stacca, ed è l'unico caso in cui il riconoscimento non ha davvero appigli.

La pagina del PDF ora calza il documento: prende il rettangolo più grande con le proporzioni dell'immagine che sta dentro un A4. Un foglio A4 esce A4, uno scontrino esce con la forma di uno scontrino, intorno a 74 per 297 millimetri, invece di diventare una striscia in mezzo a una pagina vuota per metà. Resta tutto stampabile su A4.

## Due limiti da conoscere

Il primo riguarda il destinatario. Android non permette a una pagina web di compilare il campo destinatario *e* allegare un file nella stessa operazione: è una barriera di sicurezza del sistema, non un difetto dell'app. Perciò quando tocchi **Mail** si apre Gmail (o Outlook) con il PDF già allegato, ma il campo "A" resta da riempire. Per questo l'app copia l'indirizzo negli appunti: ti basta un incollo. Su WhatsApp il pannello di condivisione mostra in cima i contatti recenti, quindi in pratica è un tocco. Se il destinatario precompilato davvero in automatico ti è indispensabile, l'unica via è un'app installata sul telefono oppure un piccolo servizio di invio lato server: dimmelo e ne parliamo.

Il secondo riguarda la memoria. Le pagine non inviate vivono solo finché l'app è aperta: se la chiudi, si perdono. È una scelta, non una dimenticanza, e serve a non lasciare documenti di lavoro sparsi nel telefono. Se preferisci che tenga una copia in Download, è una modifica di poche righe.

## Se qualcosa non va

Se il mirino non si apre o i pulsanti di invio non fanno nulla, quasi sempre l'indirizzo è `http://` invece di `https://`, oppure la pagina è aperta dentro il browser di un'altra app. Apri l'app dall'icona sulla schermata Home e il problema sparisce.

La prima volta Chrome chiede il permesso di usare la fotocamera: va concesso, altrimenti il mirino non può funzionare. Se lo neghi per sbaglio, l'app ripiega sulla fotocamera di sistema (scatti la foto con l'app Fotocamera e lei la riprende da lì, ritagliandola comunque); per ridare il permesso, tocca il lucchetto accanto all'indirizzo in Chrome e riattiva la fotocamera.

Se il riquadro giallo non compare, il bordo del foglio non si stacca abbastanza dallo sfondo: sposta il documento su un piano di colore diverso dalla carta, oppure allontanalo dal bordo del tavolo. Serve comunque che il foglio sia tutto dentro l'inquadratura, angoli compresi: se ne esce uno, l'app preferisce non proporre un ritaglio sbagliato. In ogni caso **Modifica** permette di sistemare gli angoli a mano, **Ritrova bordi** ritenta il riconoscimento sulla foto ferma, che ha più pixel e quindi più precisione, e **Tutta la foto** rinuncia al ritaglio.
