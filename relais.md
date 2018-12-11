# Relais

Ce document décrit les communications avec un relais

## Header d'un message addressé a un relais

Les relais utilisent les mêmes structures que les messages classiques
définis dans [messages.md](messages.md)

Les messages qu'un relais doit implementer `getaddr`,
`broadcast`, `getdata`, `whoami`, `whoamiack`
 
### `broadcast`

Ce message demande a un relais de notifier tout les noeuds connectés d'une transaction

| Field Size | Description | Data Type  | Comments
|------------|-------------|------------|-------------------------
| 32		 | hash		   | char[32]   | Hash de la transaction
| ?			 | transaction | transaction| Transaction a envoyer
