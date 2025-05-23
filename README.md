# NachosOS
Build a simple emulated OS

Nachos internal structure

![alt text](assets/image.png)

NachOS est un système d’exploitation écrit en C++.  
L’intérêt est de pouvoir manipuler directement les composants d’un OS (threads, mémoire, IO)  en testant et déboguer notre code.

Le projet est compsé de deux parties:

- 🔹 TP1 : les **mécanismes de synchronisation entre threads**
- 🔹 TP2 : la **gestion des communications série** avec l’ACIA (console asynchrone)
ans ce premier TP, nous avons implémenté les mécanismes de **synchronisation entre threads** dans NachOS.  
Ce travail nous a permis de comprendre comment fonctionnent :
- les **sémaphores**,
- les **verrous (locks)**,
- les **variables de condition**,
- ainsi que la création, le démarrage, l’attente, et la terminaison de threads.

---

##  Fichiers modifiés et fonctions implémentées

---

###  `kernel/synch.h` et `kernel/synch.cc`

####  `Semaphore::P()` (wait)
- Désactive les interruptions
- Décrémente le compteur du sémaphore
- Si le compteur devient négatif, le thread courant est bloqué (ajouté à `waiting_queue`)
- Rétablit le statut des interruptions

####  `Semaphore::V()` (signal)
- Désactive les interruptions
- Incrémente le compteur
- Si des threads sont en attente, le premier est réveillé
- Rétablit le statut des interruptions

---

####  `Lock::Acquire()`
- Si le verrou est libre, le thread courant le prend
- Sinon, il est ajouté à la file d’attente et mis en sommeil

####  `Lock::Release()`
- Si le thread courant possède le verrou, il le libère
- Si d’autres threads attendent, on en réveille un et on lui donne le verrou

####  `Lock::isHeldByCurrentThread()`
- Renvoie vrai si le thread courant détient le verrou

---

####  `Condition::Wait()`
- Le thread courant est ajouté à la file d’attente de la condition et endormi

####  `Condition::Signal()`
- Réveille un thread en attente sur la condition (s’il y en a)

####  `Condition::Broadcast()`
- Réveille tous les threads en attente sur la condition

---

###  `kernel/thread.cc` et `kernel/thread.h`

####  `Thread::Start()`
- Initialise le contexte du thread
- Alloue une pile
- Ajoute le thread dans la liste des threads actifs (`g_alive`)
- L’ajoute dans le scheduler (état prêt)

####  `Thread::Finish()`
- Utilisé à la fin d’un thread
- Affiche un message de debug
- Met le thread en sommeil (n’est pas complètement supprimé dans notre version)

####  `Thread::Yield()`
- Le thread courant cède le processeur volontairement à un autre thread prêt
- S’il y a un autre thread prêt, il est planifié et on effectue un `SwitchTo`

####  `Thread::RestoreProcessorState()`  
- Restaure les registres du processeur à partir du contexte du thread
- Nécessaire pour que le thread reprenne son exécution correctement

####  `Thread::SaveProcessorState()`  
- Sauvegarde les registres du thread courant avant un switch
# TP2

L’objectif du TP2 était d’implémenter un driver de communication série (ACIA) dans NachOS.  
Ce driver permet :
- l’**envoi de messages vers la console caractère par caractère
- la réception de messages depuis la console
- en utilisant des interruptions

---



##  Fichier : `drvACIA.cc`  
###  Fonctions implémentées

---

###  `TtySend(char* buff)`
- Copie le message dans `send_buffer`
- Envoie le premier caractère avec `PutChar()`
- Se bloque ensuite avec `send_sema->P()` jusqu’à ce que tout soit envoyé
- C’est `InterruptSend()` qui envoie les caractères suivants

---

### `InterruptSend()`
- Appelée automatiquement après chaque caractère envoyé
- Incrémente `ind_send` et envoie le caractère suivant
- Si on atteint la fin (`'\0'`), on appelle `send_sema->V()` pour réveiller le thread bloqué

---

### 🔸 `TtyReceive(char* buff, int lg)`
- Se bloque avec `receive_sema->P()` jusqu'à la réception complète d’un message
- Copie les caractères depuis `receive_buffer` vers `buff`
- Ajoute un `'\0'` à la fin
- Retourne le nombre de caractères copiés

---

### 🔸 `InterruptReceive()`
- Appelée à chaque réception d’un caractère
- Stocke le caractère dans `receive_buffer[ind_rec]`
- Si c’est la fin (`'\0'`), on termine la chaîne et appelle `receive_sema->V()`

---

