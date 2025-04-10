MODELLAZIONE DATI <br>
<br>
Dopo un’iniziale esplorazione delle tabelle, sono state selezionate le seguenti: <br>
-	olist_orders_dataset            <br>
-	olist_order_items_dataset  <br>
-	olist_customers_dataset (rinominata customers) <br>
-	olist_order_review_dataset (rinominata reviews) <br>
-	olist_product_dataset <br>
-	product_category_name_translation <br>
<br>
Dopo aver effettuato attività di pulizia dei dati — tra cui l'utilizzo di funzioni come Trim, la rimozione di duplicati, la verifica dei formati e l’eliminazione di colonne non significative ai fini analitici — sono state implementate le seguenti modifiche: <br>
-	In olist_orders_dataset è stata aggiunta la colonna calcolata delivery_time, che misura i giorni di consegna a partire dalla data dell’ordine. <br>
-	Nella tabella customers, si è notato che la colonna customer_id si riferisce a ciascuna transazione, mentre customer_unique_id al cliente effettivo. Inoltre, customer_state è stata sostituita con i nomi delle regioni per esteso (basati su dati da Wikipedia), per evitare confusioni con omonimie internazionali. <br>
-	Le tabelle olist_orders e olist_order_items sono state unite in un’unica tabella chiamata orders. In questa, la colonna delivery_time è stata rimpiazzata da delivery_time_grouped, che raggruppa i valori in tre classi: <7, 7 to 10, >10. Questa scelta facilita l’utilizzo dei filtri. Il merge è stato effettuato per semplificare lo schema delle relazioni, trattandosi di un dataset statico. <br>
-	Nella tabella reviews, i duplicati su order_id sono stati rimossi in quanto legati a modifiche successive a una stessa recensione. <br>
-	Infine, è stato effettuato un merge tra olist_product_dataset e product_category_name_translation, ottenendo la tabella products_&_categories, contenente gli ID prodotto e il nome della categoria in inglese. <br>
<br>
<br>
<br>
DAX E MISURE <br>
<br>
-	tabella calendar :  create per una gestione corretta delle analisi temporali <br>
<br>
orders: <br>
-	_order_purchase_date : ottenuta tramite DATEVALUE a causa di problematiche sulla colonna originale order_purchase_timestamp <br>
-	_tot_orders : numero di ordini, calcolato con DISTINCTCOUNT(order_id) <br>
-	_tot_orders_PY : calcolata con SAMEPERIODLASTYEAR e condizionato da HASONEVALUE affinché sia visibile nei visual soltanto quando è selzionato un anno in modo da evitare aggregazioni insensate (se non si filtra per un anno non ha senso comparare i trend con “l’anno passato”) <br>
-	_tot_revenue : calcolato con SUMX su price e freight_value <br>
-	_tot_revenue_PY : stessa formula di tot_orders_PY <br>
-	_var_order : creata con DIVIDE (([_tot_orders]-[_tot_orders_PY]) , orders[_tot_orders_PY], "--") <br>
-	_var_revenue : come _var_order <br>
<br>
customers: <br>
-	_max_region_orders : misura creata per ottenere la regione con più ordini CALCULATE(FIRSTNONBLANK('customers'[customers_state], 1), 
                     TOPN(1, SUMMARIZE(customers, 'customers'[customers_state], "Total Orders", [_tot_orders]), [_tot_orders], DESC)) <br>
-	_max_region_revenue : come sopra <br>
-	_max_region_review : come sopra <br>
<br>
reviews: <br>
-	_review : misura create con DISTINCTCOUNT per comodità di calcolo in _max_region_review <br>
<br>
<br>
<br>
DETTAGLI SUI VISUAL <br>
<br>
La palette cromatica è stata definita in coerenza ai codici colore del sito web aziendale e dell’immagine del logo. <br>
La lingua inglese è stata una scelta dettata dal fatto che si tratta di un’azienda brasiliana. <br>
<br>
I filtri di ogni pagina sono posizionati sulla sinistra della canvas. <br>
In basso a destra si trovano i pulsanti di navigazione tra le pagine. <br>
<br>
L’aggiunta di una scheda relativa alla regione permette anche di visualizzare il nome di quest’ultima se si filtra attraverso la mappa. <br>
<br>
Dove opportuno, sono state aggiunte descrizioni comando. Per esempio, nei grafici mostra valori percentuali come la variazione rispetto l’anno precedente oppure sulle icone per descrivere le loro azioni. <br>
<br>
-	Nelle prime due pagine (Orders e Revenue) i grafici mostrano misure aggregate per mese per poter visualizzare eventuali trend stagionali. Se si filtra per anno, verrà aggiunto al grafico l’andamento dell’anno precedente a quello selezionato. <br>
-	In Revenue, sono stati esclusi gli ordini con status canceled o unavailable, non apportando ricavi effettivi. <br>
-	La pagina Product introduce una navigazione tramite icone, che rimandano ad analisi mirate su ordini e profitti. È possibile filtrare le categorie attraverso i grafici ad anello. <br>
-	Nella pagina Rating vengono rimosse le schede relative ai totali e alla variazione annua per fare spazio ad un grafico a linea che analizza l’andamento nel tempo della media dei voti di recensione.
-	Infine, nella pagina Insight, cliccando sull’icona del robot è possibile interagire con la funzione Q&A, con domande suggerite e sinonimi già predisposti. La pagina inizialmente si presenta con una guida all’utente e ,una volta selezionata una delle icone, si passa all’analisi relativa.
