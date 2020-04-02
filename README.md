Easy Doctrine  
===============

Clase para paginar Entidades sencillas de Doctrine

## Instalacion

```
composer require antoniosam/easy-doctrine
```
## Uso

La clase compila metodos genericos para realizar una paginacion sencilla
```
$paginar = new EasyDoctrine($em,Entity::class);
$paginar->setPage(1);
$paginar->order('name','ASC');
$paginar->execute();

$info = $paginar->getResult();
```
#### setPage
Si al mentodo **setPage** se ingresa la pagina **0**, esto probocara que no se pagine la respuesta y se ejecute la consulta sin **limit** y **offset**
## Respuesta
Despues de ejecutar la consulta la respuesta es una clase EasyData datos obtenidos
```
(EasyData) 
    $result->totalrecords = 100;
    $result->data = DoctrineArray;
    $result->page = 1;
    $result->pages = [1,2,3];
    $result->totalpages = 3;
    $result->itemsbypage = 20;
    $result->firstrecord = 1;
    $result->lastrecord = 10;

```

## Funcionamiento 

El metodo **where($campo,$comparacion,$valor)** internamente ejecuta una instruccion 
```
->andwhere('campo comparacion :contador')->setParameter('contador',$valor);
ej.
->andwhere('a.name = :donde1')->setParameter('donde1','Sara');
```
El metodo **search($valor,$campos)** internamente ejecuta una instruccion 
```
->andwhere('campo1 LIKE buscar OR campo2 LIKE buscar OR campo_n LIKE buscar')->setParameter('buscar',$valor);
ej.
->andwhere('a.name LIKE :buscar OR a.email LIKE :buscar')->setParameter('buscar','Sara'.'%');
```
El metodo **order($columna,$orden)** internamente ejecuta una instruccion 
```
->orderby('columna','orden');
ej.
->orderby('a.name','ASC');
```

## Ejemplo Completo 
Primero recuperamos la informacion que recibe el controlador
```
$pagina = $request->get('pagina', 1);
$columna = $request->get('columna', null);
$orden = $request->get('orden', null);
$buscar = $request->get('buscar', null);
```
Asignamos el tipo de ordenamiento

La lista de campos deben ser un arreglo que contenga **solo** las propiedades de la entidad 
```
$campos = ['id','name' ,'lastname', 'email', 'gender','active', 'created_at'];
$col =  (!is_null($columna) && $columna != '')?$campos[$columna]:'id';
$ord = (!is_null($columna) && $columna != '')?$orden:'ASC';
```
Instanciamos y agregamos la condiciones necesarias
```
$em = $this->getDoctrine()->getManager();
        
$paginar = new EasyDoctrine($em,Entity::class);
$paginar->setPage($pagina);
$paginar->where('active','=',true);
if(!empty($buscar)){
    $paginar->search($buscar,['name','email']);
}
$paginar->order($col,$ord);
$paginar->execute();

$info = $paginar->getResult()

```

## Left Join
La clase permite agregar una relacion y utilizarla para ordenar o aplicar alguna condicion
se debe ejecutar primero el metodo **leftJoin($campo,$indicador)** indicando el campo que tenga relacion y un indicador diferente de **a** o **b** 


```
$campos = ['municipio','delegacion','direccion', 'numero', 'colonia'];
$col =  (!is_null($columna)  $columna != '')?$campos[$columna]:'direccion';
$ord = (!is_null($columna)  $columna != '')?$orden:'ASC';
 
$paginar = new EasyDoctrine($em,Casa::class);
$paginar->setPage($pagina);
if(!empty($buscar)){
    $paginar->search($buscar,['direccion','colonia']);
}
if($col == 0){
    $paginar->leftJoin('delegacion','d');
    $paginar->leftJoin('municipio','m','d');
    $paginar->ordenar('nombre','ASC,'m');
}else{
    $paginar->ordenar('campo','ASC');
}
$paginar->execute();

```
De la entidad Casa buscamos la relacion casa -> delegacion

Despues bucamos la relacion delegacion -> municipio

Y ordenamos

```
->leftJoin('a.delegacion', 'd')
->leftJoin('d.municipio', 'm')
->orderby('m.nombre', 'ASC')
```
