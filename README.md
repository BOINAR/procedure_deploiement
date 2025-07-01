# procedure_deploiement
Guide pour effectuer des déploiement sur un serveur vps


### Se connecter à un serveur distant
```typeScript
 ssh user@ip_du_serveur
```


#### Générer une paire de clé
```typeScript
ssh-keygen -t rsa -b 2048
```

#### Trasnférer la clé publique vers serveur distant

```
ssh-copy-id user@ip_du_serveur

