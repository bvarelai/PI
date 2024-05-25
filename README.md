#  Programacion Integrativa (2023-2024)
## Apuntes sobre shell scripting
- BREs : Expresiones regulares basicas.
**Ejemplo** : [...]  \(\)  \n  \{n,m\} 
- EREs : Expresiones regulares extendidas
**Ejemplo** : [...] {n,m} + ? | ()
Los caracteres comunes en ambas son `\` `.` `*` `^` `$`.
El `^` es el inicio de una lineas o string pero en BREs solo es metacaracter al principio de una expresion regular
El `$` es el fin de una lineas o string pero en BREs solo es metacaracter al final de una expresión regular
El `\n` hace referencia a lo que esta entre parentesis (la repite)
El `{n,m }` se repite hace que se repita entre n y m veces 


Con EREs se pueden agrupar expresiones regukares usando `( y )` y tambien se pueden
utilizar difrenetes alternativas como (0|1). Ademas los `^` y `$` son metacaracteres en EREs 

### Lo basico
#### Comando sed
- Comando con el que sustituyes valores en ficheros de texto
- `sed 's/foo/FOO/gi' <fichero>` : sustituye los foo por FOOS sin sensibilidad (no es sensible a mayusculas)
- `sed '2,3s/foo/FOO/gi' <fichero>` : lo mismo pero afecta a ese rango de lineas. Si ponemos 2,3! afecta a la 1 y 4

Solucion al ejemplo : sed '2,3!s/foo/FOO/' file_sed (ESTA MAL)

#### Comando grep
- Comando para buscar lineas en ficheros
- `grep 'cadena' <fichero1>`. Si es fichero* busca ficheros que empiezen por fichero, -r para ser recursivo
- `grep -c` : cuenta el nº de lineas en las que aparece el patron
- `grep -o` : muestra parte de la linea en la que esta el patron
- `grep -l` : muestra ficheros en los que esta el patron
- `grep -n` : muestra el nº de linea
- `grep -w` : busca solo palabras completas que coinciden con el patron
- `grep -i` : no sensible
- `grep -v` : invertir lo que quieres hacer  

**Ejemplo** : `grep -on "bar.*bar" file_sed.`  
Esto va a mostrar las partes de las lineas (con su numero) en donde aparezca dos palabras bar  
y entre ellas puede haber cualquier tipo de caracter `.` cero o mas veces `*` 

- `grep "h[oò]la"` : devuelve las lineas que entre la h puede haber o con tilde o sin ella
- `grep "\.\(hola\)\1" cap2` : delvuelve ".holaholahol"
- `grep "\(hola_\)\1  cap2"` : te busca cualquier cosa con hola


Solucion al ejemplo : grep -vwn "foo" file_sed

#### Comando find
- Lista ficheros y directorios de la ruta actual
- `find -name/-iname` : por nombre fichero/directorio
- `find -mtime n` : fecha de modificacion
  **Ejemplo** : `find. -type f mtime 3`
- `find -empty` : ficheros vacios
- `find -not` : negar la opciones
- `find -exec command` : ejecutar comando
- `find -user/-uid/-group`

### Cosas chungas
- `find -type f | grep "\.so$"` : Buscar ficheros y mostrar las lineas que acaben en .so
La `\` es para indicar que el `.` no es un caracter especial. Por lo que `"\.so$"` acabaria en .so y `".so$"` acabaria en so
- `find -type f | sed -e "s  /  .*\/\(.*\)  /  \1 /"` : Eliminas la ruta de los ficheros para que en el caso de que haya directorios con el nombre
que buscas no los muestre. Asi solo muestra ficheros, el `\/` es para indicar un caracter `/` normal.  
Buscas texto que empiece con cualquier caracter seguido de `/` y siga con cualquier caracter, el resultado que devuelve el `\(...\)` se copia en el `\1`.  
Siempre el `.*` va a ser de el mayor tamaño posible para que el `\1` sea solo el fichero.
- Hacerlo asi no es eficiente ya que perdemos la informacion de las rutas de los ficheros y nos pueden aparecer ficheros repetidos.
Por ello usaremos comandos como `find -type f | grep "video[^/]*$"`, donde el `[^/]` hace referencua a cero o mas caracteres cualquiera execpto el `/`.
De manera que buscamos cosas con nombre video que no tenga el `/` y ademas acaben con cualquier caracter. Otra manera es con `find -type f -regextype posix-basic \ -regex ".*video[^/]*"`,
donde se indica un `.*` al principio de video por que ya que si no find aceptara lienas que comiencen por video.

### Escribir scripts
#### Parseo de argumentos
- Si queremos pasar/parsear argumentos con alias debemos usar getopts.
```bash
#!/bin/bash
while getopts "ax:" opt; do    ## el "ax:" indica en este caso las opciones (tambien sirve para indicar si existen parametros) y opt es la variable a iterar
 case $opt in
  a) echo "-a was triggered" ;;                       ## sin parametros
  x) echo "-x was triggered with option $OPTARG" ;;   ## con parametros
  \?) echo "Invalid option: -$OPTARG" ;;
 esac
done
```
`"ax"` indica que ni `a` ni `x` llevan parametro detras
`"ax:"` indica que solo `x` lleva parametro detras
`"a:x:"` las dos llevan
- Tambien tenemos getopt para lo mismo pero con argumentos largos
```bash
# DEPENDENCY: getopt command tool
TMP=‘getopt --options ab:c:d --long arga,argb:,argc:,argd,arge \
             -n "$SCRIPTNAME"\
             -- "$@"‘ ## pasar los argumentos tal cual a getopt
# Note the quotes around ‘$TMP’: they are essential!
eval set -- "$TMP"
```
#### Atrapar señales
- Para atrapar señales usamos el comando trap
```bash
#!/bin/sh
## remove al temporal files created
cleanup() {
  echo "caught signal: cleaning up..."
  rm -rf /tmp/*.tmp
  echo "done cleanup! quitting..."
  exit 1;
}
## catch signals and execute cleanup
trap cleanup 1 2 3 6

## main loop
for i in *
do
  sed -e "s/FOO/BAR/g" $i > /tmp/${i}.tmp && mv /tmp/${i}.tmp $i
done
```

## Apuntes sobre pandas
