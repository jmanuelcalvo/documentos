#Lanzar una aplicacion desde un git privado

Primero crear un proyecto y lanzar una aplicacion
```
oc new-project carrito
oc new-app php~https://github.com/jmanuelcalvo/shopping.git
```

Luego se debe crear recurso de tipo secret dentro del proyecto, este puede ser con los datos usuario y password que usas en git
```
oc secrets new-basicauth github-credentials --username=jmanuelcalvo --password=XXXXXX
```

Una vez creado el recurso, lo mas seguro es que el proceso de build fallo gracias a que no se pudo realizar el git clone, por esta razon se debe adicioanr al buildconfig los datos del git creados en el paso anterior
```
oc patch buildConfig shopping -p '{"spec":{"source":{"sourceSecret":{"name":"github-credentials"}}}}'
```

y una vez adicionado este parametro, se indica al build que se ejecute una nueva version
```
oc start-build bc/shopping
```

Por ultimo puede validar que la aplicacion se cree de manera correcta
