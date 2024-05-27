#  Programacion Integrativa (2023-2024)
## Apuntes sobre shell scripting
- BREs : Expresiones regulares basicas.  
**Ejemplo** : [...]  \(\)  \n  \{n,m\}   
- EREs : Expresiones regulares extendidas    
**Ejemplo** : [...] {n,m} + ? | ()  
Los caracteres comunes en ambas son `\` `.` `*` `^` `$`.  
El `^` es el inicio de una lineas o string pero en BREs solo es metacaracter al principio de una expresion regular.  
El `$` es el fin de una lineas o string pero en BREs solo es metacaracter al final de una expresión regular.  
El `\n` hace referencia a lo que esta entre parentesis (la repite).  
El `{n,m }` se repite hace que se repita entre n y m veces.  
Con EREs se pueden agrupar expresiones regukares usando `( y )` y tambien se pueden  
utilizar diferentes alternativas como (0|1). Ademas los `^` y `$` son metacaracteres en EREs.
### Lo basico
#### Comando sed
- Comando con el que sustituyes valores en ficheros de texto
- `sed 's/foo/FOO/gi' <fichero>` : sustituye los foo por FOOS sin sensibilidad (no es sensible a mayusculas).
- `sed '2,3s/foo/FOO/gi' <fichero>` : lo mismo pero afecta a ese rango de lineas. Si ponemos 2,3! afecta a la 1 y 4.  
##### Solucion al ejemplo
`sed '2,3!s/foo/FOO/' file_sed` (ESTA MAL)  
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
- `grep "h[oò]la"` : devuelve las lineas que entre la h puede haber o con tilde o sin ella.  
Mejor es usar `"[[=o=]]"`, ya que busca la letra para cualquier idioma.  
```bash
[u@host:/usr]$ export V ="[=a=][=e=][=i=][=o=][=u=]"
[u@host:/usr]$ . ~/ej2.sh " ^[^$V]*[$V][^$V]*[$V][^$V]*[$V][^$V]*$"
```
Esto busca cadenas que:  
1. Empiecen con cualquier número de caracteres que no sean vocales ([^$V]*).  Tengan una vocal ([$V]).  
2. Continúen con cualquier número de caracteres que no sean vocales ([^$V]*). Tengan otra vocal ([$V]).  
3. Continúen con cualquier número de caracteres que no sean vocales ([^$V]*). Tengan otra vocal ([$V]).  
4. Terminen con cualquier número de caracteres que no sean vocales ([^$V]*).  
- `grep "\.\(hola\)\1" cap2` : delvuelve ".holaholahol"
- `grep "\(hola_\)\1  cap2"` : te busca cualquier cosa con hola  
##### Solucion al ejemplo
`grep -vwn "foo" file_sed`  
#### Comando find
- Lista ficheros y directorios de la ruta actual
- `find -name/-iname` : por nombre fichero/directorio
- `find -mtime n` : fecha de modificacion  
  **Ejemplo** : `find. -type f mtime 3`
- `find -empty` : ficheros vacios
- `find -not` : negar la opciones
- `find -exec command` : ejecutar comando
- `find -user/-uid/-group`
#### Comando basename
- Le pasas una ruta y elimina del arbol de directorios el sufijo de esta
Teniamos un problema y es que con la expresion `eje1.sh ".\/[a-zA-Z]*$"` buscabamos los ficheros que solo contuvieran letras en su nombre,  
y es que es limitado. Una de las opciones de arreglo es `eje1.sh "[a-zA-Z]*[^/]*$"`, pero en este caso evitando que aparezca el `/` pero no numeros 
Para poder pillar la `/` mejor podemos usar `basename include/malloc.h`. Esto devuelve malloc.h pero no es util ya que hay que pasarle la cadena como parametro
#### Comando xargs
- Construye y ejecuta comandos a partir de la entrada estandar
Podemos usar este comando como solucion al anterior `find -type f|xargs -n 1 basename| grep "^[a-zA-Z]*$"` donde -n 1 significa que se debe llamar a basename por cada
parametro que reciba por entrada estandar. Pero en esta solucion al no tener la ruta pueden aparecer duplicados
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
#### Script complejos
```bash
[user@host:/usr]$ for i in $( find - type f ); \
do \
N="$(basename $i) "
if grep -q " ^[a-zA-Z]*$" <<< "$N " ; then   ## El <<< es para pasarle el contenido de la variable N a grep
echo "$i " ; \
fi \;
done
./include/X11/bitmaps/mailfullmsk
./include/X11/bitmaps/scales
./include/X11/bitmaps/Excl
./include/X11/bitmaps/mensetmanus
./include/X11/bitmaps/black```
```
- El termino `"EOF"` es para que el interprete usado no intente hacer ninguna sustitucion en el cuerpo de HEREDOC  
(que es una manera de mostrar texto por salida estandar sin hacer saltos de linea). Por ejemplo:
```shell
$ tr a-z A-Z <<IDENTIFICADOR
> Hola a
> todos, espero que
> "se encuentren muy bien";
> Saludos!!
> IDENTIFICADOR
```
con salida:  
```
HOLA A
TODOS, ESPERO QUE
"SE ENCUENTREN MUY BIEN";
SALUDOS!!
```
## Apuntes sobre pandas
### Definir un Dataframe
```python
data = { 'comunidad': [ 'Galicia', 'Galicia', 'Galicia', 'Asturias', 'Asturias' ],
         'año': [2000, 2001, 2002, 2001, 2002],
         'población': [2731900, 2732926, 2737370, 1075329, 1073971] }
frame = DataFrame(data)
frame
```
### Mover las columnas
```python
Dataframe(data,columns=['poblacion','comunidad',año]) 
```
### Cambiar los indices numericos por letras
```python
frame2 = DataFrame(data, columns=['año', 'comunidad', 'población', 'deuda'],index=['uno', 'dos', 'tres', 'cuatro', 'cinco']) 
```
### Recuperar una columna
```python
frame2['comunidad']
frame2.comunidad
```
### Recuperar una fila
```python
frame2.loc['tres']
frame2['comunidad'].iloc[3] 
```
### Modificar un dato
```python
frame2.deuda = 3000
```
## Reindexar (mover indices)
```python
frame2 = frame2.reindex(['dos', 'uno', 'tres', 'cinco', 'cuatro'])
```
### Añadir una columna/fila** :

- Asignando una columna que no existe
```python
frame2['elecciones'] = False
del frame2['elecciones'] ##Borras la columna
frame2 = frame2.drop(1) ## Eliminar una fila por su indice
frame2 = frame2.drop('elecciones', axis=1) ## Eliminar columna por su nombre
```
### Sumar totales por columna/fila** :  
```python
frame2['deuda'].sum() ##por columna
frame.iloc[2].sum() ##por fila
frame2['deuda'].mean() ##Hacer media por columna
```
### Renombrar una columna
```python
df.rename(columns={"Tipo consumo": "Tipo", "Consumo (kwh)": "kwh"}, inplace=True)
```

### Crear subtablas   

### Guardar el Dataframe en un archivo**
```bash
import pickle
with open('df.pkl','wb') as f:  ##f es el archivo y wb el modo de escritura (binario)
  pickle.dump(df,f)
```
### Indexar un Dataframe usando un patron booleano alternante
```bash
precio[[false,true]*(744//2)]
```
### Sumar una columna filtrando previamente por filas
```bash
croissants = df[df.Item == 'Croissant']
ganancias_totales_croissants = croissants['Precio Unitario'].sum()
### Ganancias totales de la venta de cada producto
ganancias_totales = df.groupby('Item')['Precio Unitario'].sum()
```
## Tema orientado a objetos 
### Ejemplo (Solucion 1º ejercicio)
```python
class Number():
   unit = "Kilograms"
   value = 0
   def __init__(self, value : int):
      self.value = value

def main() :
    n1=Number(10)
    print(str(n1.value)+" "+Number.unit)
    Number.unit="Grams"
    n2=Number(20)
    print(str(n1.value)+" "+Number.unit)
    print(str(n2.value)+" "+Number.unit)

if __name__ == "__main__":
    main()
```
- El metodo `__init__` es el constructor, `__del__` el destructor, `__add__` el de Suma , `__or__` es OR
y `__repr__` es para imprimir como str. Todos estos son los **metodos magicos**.
```python
def __init__(self,x):
    self.x=x
def __add__(self,other):
    return MyClass(self.x+other.x)
def __repr__(self):
    return "My value of x is: "+str(self.x)
```
- Los **metodos estaticos** son aquellos que podemos llamar a traves del el nombre de la clase o de una instancia,
los **metodos de clase** son inguales que estos pero estan ligados a una clase
```python
class MyClass:
  __ninstances=0
  def __init__(self):
    type(self).__ninstances+=1
  @staticmethod
  def nInstances():
    return MyClass.__ninstances
```
```python
  @classmethod
  def nInstances(cls):
    return cls.__ninstances
```
```python
  @property
  def x(self):
    return self.__x
  @x.setter 
  def x(self,x):
    if x<0:
      self.__x=0
    elif x > 10:
      self.__x=10
    else:
      self.__x=x
```


