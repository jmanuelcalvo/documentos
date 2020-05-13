# Configuracion de Ansible Tower contra Active Directory

Para este ejercicio se realiza la configuracion de un Directorio Activo que contiene usuarios y grupos sobre la carpeta Users

Desde una de las maquinas de Ansible Tower se valida la conectividad con el siguiente comando:
```
[root@localhost ~]# ldapsearch -x -LLL -h 10.42.0.50 -b dc=jmanuelcalvo,dc=com -D jmanuel@jmanuelcalvo.com -w ABCde123.
```


## Parametros de Configuracion de Ansible Tower con Active Directory

![Ansible Tower](images/ATldap1.png)


### LDAP USER SEARCH
```bash
[
 "DC=jmanuelcalvo,DC=com",
 "SCOPE_SUBTREE",
 "(sAMAccountName=%(user)s)"
]
```


### LDAP GROUP SEARCH
```
[
 "CN=s_cloudforms_admin,CN=Users,DC=jmanuelcalvo,DC=com",
 "SCOPE_SUBTREE",
 "(objectClass=group)"
]
```

### LDAP USER ATTRIBUTE MAP
```
{
 "first_name": "givenName",
 "last_name": "sn",
 "email": "userPrincipalName"
}
```

### LDAP GROUP TYPE PARAMETER
```
{}
```

### LDAP USER FLAG BY GROUP
```
{
 "is_superuser": [
  "CN=AT_ADMIN,CN=Users,DC=jmanuelcalvo,DC=com"
 ]
}


```
