### Linux swap memory
Linux divide la RAM in aree di memoria chiamate pagine. Lo **swapping** è il processo mediante il quale una pagina di memoria viene copiata in uno spazio preconfigurato sul disco rigido, chiamato spazio di **swap**, per liberare dalla memoria. Le dimensioni combinate della memoria fisica e dello spazio di swap è la quantità di memoria virtuale disponibile. Lo swapping è necessario per due motivi importanti:

1. In primo luogo, quando il sistema richiede più memoria di quella fisicamente disponibile, il kernel sposta le pagine meno utilizzate nello spazio di swap e concede l’utilizzo della memoria ram all’applicazione corrente (processo) che in quel momento richiede la memoria.
2. In secondo luogo, un numero significativo di pagine utilizzate da un’applicazione durante la sua fase di avvio possono essere utilizzate solo per l’inizializzazione del sistema e poi mai più usate.

Il sistema è in grado di usare quindi lo swap su quelle pagine e di liberare la memoria per altre applicazioni o addirittura per la cache su disco. Tuttavia, lo swapping ha un rovescio della medaglia. Rispetto alla memoria RAM, i dischi sono molto più lenti. La velocità della memoria è misurata in nanosecondi, mentre quella dei dischi in millisecondi, dunque l’accesso al disco è decine di migliaia di volte più lento rispetto alla memoria ram. Più operazioni di swapping che si verificano, più lento il vostro sistema sarà. A volte un eccessivo swapping crea dei colli di bottiglia, poichè si verifica una particolare situazione: una pagina viene messa nello swap e poi portata in ram molto velocemente ed in modo continuativo. In tali situazioni il sistema lotta per trovare della memoria libera e mantenere le diverse applicazioni in esecuzione nello stesso momento. In questo caso, solo l’aggiunta di RAM più aiutare la stabilità del sistema stesso.

Linux ha due forme di spazio di swap: la partizione di swap e il file di swap. La partizione di swap è una sezione indipendente del disco fisso, utilizzati esclusivamente per lo swap, nessun altro può risiedere lì. Il file di swap è un file speciale che risiede nel filesystem tra il sistema e file di dati. Per vedere com’è fatto e dove è ubicato lo spazio di swap che si possiede, si utilizza il comando ``swapon``.

```
# swapon -s
Filename                                Type            Size    Used    Priority
/dev/dm-0                               partition       4079612 0       -1
```

Ogni riga elenca una partizione di swap separata utilizzata dal sistema. Una particolarità dello swap su linux è che, se montare due (o più) spazi di swap (preferibilmente su due dispositivi differenti) con la stessa priorità, linux divide le sue attività di swapping tra di loro. Questo si traduce in un incremento notevole delle prestazioni. Per aggiungere una partizione di swap per il vostro sistema, è necessario però prima di prepararla.

### Add a swap area as a file
```
dd if=/dev/zero of=/var/swapfile bs=1M count=2048
chmod 600 /var/swapfile
mkswap /var/swapfile
echo /var/swapfile none swap defaults 0 0 | sudo tee -a /etc/fstab
swapon -a
```

