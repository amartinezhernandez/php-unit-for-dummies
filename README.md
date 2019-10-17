# Unit tests for dummies

Esta es una guía básica de unit test para dummies hecha por un dummy D:

## Estructura de un test

### SetUp

Este método se ejecuta antes de ejecutar cualquier test. Podemos usarlo para inicializar casos de uso, repositorios, etc...

```
public function setUp()
{
    parent::setUp();
    // Inicializamos aquí lo que vayamos a necesitar en todos los test, casos de uso, mocks, etc...
}
```

### TearDown

Este método se ejecuta después de ejecutar cualquier test.

```
public function tearDown()
{
    array_map('unlink', glob($this->filesPath.'*.*'));
    rmdir($this->filesPath);
}
```

### Test

Este método será nuestro test en sí. Para indicar a PHP Unit que tiene que ejecutar este método, le podemos poner un comentario con @test o llamar al método testNOMBREDELMÉTODO: 

```
public function testWorks()
{
    $data['number'] = 1;
    $this->assertEquals(1, $data['number']); // true;
    $this->assertNotNull(1); // true;
}

/**
 *   @test
 **/
public function thisAlsoWorks()
{
    $data['number'] = 1;
    $this->assertEquals(1, $data['number']); // true;
    $this->assertNotNull(1); // true;
}
```

## Mocks

Con PHP Unit podemos "mockear" (crear un doble creado a partir de la clase para poder controlar sus funcionalidades) cualquier clase. Más info: https://phpunit.readthedocs.io/es/latest/test-doubles.html.
Al realizar un test, podemos instanciar un elemento con la clase que requiera o un mock de la misma.

### Ejemplo 

```
$this->repository = $this->getMockBuilder(MyRepository::class)
            ->disableOriginalConstructor()
            ->disableOriginalClone()
            ->disableArgumentCloning()
            ->disallowMockingUnknownTypes()
            ->getMock();

$this->useCase = new MyUseCase(
    $this->repository
);
```

### Funciones de los mock

#### Method

Indica el método al que se le aplica la acción que usemos.

En el siguiente ejemplo, al ejecutar el caso de uso anterior, si llama en el repositorio a la función 'list', ésta devolverá un array vacío:
```
$this->repository->method('list')->willReturn([]);
```

#### Expects

##### once(), exactly(X)
Si le pasamos la función expects con el parámetro once o exactly, le estaremos diciendo al test que haga una comprobación de que la función a la que se lo especifiquemos se ejecute X veces:
```
$this->repository->expects($this->once())->method('list'); // El método se llamará una vez en el caso de uso
$this->repository->expects($this->exactly(5))->method('list'); // El método se llamará 5 veces en el caso de uso
```

##### at(X)
Si le pasamos la función expects con el parámetro at, le estaremos diciendo al test que lo que le pasemos a la función solo debe comprobarse o ejecutarse cuando se ejecute la función en el índice X:
```
$this->repository->expects($this->at(0))->method('list')->willReturn([]); // El método devolverá [] cuando se ejecute por primera vez.
$this->repository->expects($this->at(1))->method('list')->willReturn(null); // El método devolverá null cuando se ejecute por segunda vez.
```

### With
Usaremos la función with para indicar al test los parámetros con los que se ejecutaría la función:
```
// Suponemos que el método list tiene un parámetro limit que le viene desde el caso de uso
$this->repository->method('list')->with(20)->willReturn(true); // true;
$this->repository->method('list')->with(10)->willReturn(true); // false; (no es el valor que debería llegar)

$this->useCase->execute($limit = 20);
```
También podemos pasarle al with una función que compruebe que el parámetro pasado cumple algún requisito:
```
$this->repository->method('list')->with($this->greaterThan(10))->willReturn(true); // true;

$this->useCase->execute($limit = 20);
```

### WillReturn
Usaremos la función willReturn para indicarle al método que debe devolver al ejecutarse la función:
```
$this->repository->method('list')->willReturn([]); // El método list devuelve []
```

### WillThrowException
Usaremos la función willThrowException para hacer que el método devuelva una excepción del tipo que le indiquemos:
```
$this->repository->method('list')->willThrowException(new ConnectionRefusedException());
```
            
## Comprobaciones

### Asserts

Nos servirán para comprobar que un valor cumple el requisito que le estamos especificando:
```
$this->repository->method('list')->with(20)->willReturn([]);

$result = $this->useCase->execute(20);
$this->assertEquals([], $result); // true;
$this->assertCount(0, $result); // true;
...
```

### Expect exception/exception message
Si queremos comprobar si el método lanza una excepción, podemos indicarselo ANTES de ejecutarlo:
```
$this->repository->method('list')->willThrowException(new ConnectionRefusedException());

$this->expectException(ConnectionRefusedException::class);
$this->expectExceptionMessage('Connection to database refused');
$result = $this->useCase->execute(20);
```

# FAQ

- Pero, con la explicación de este manual aún no me entero de cómo va la cosa, explícamelo mejor:

![](https://media.tenor.com/images/e55a7959ddd26f70bbdd691d462bb739/tenor.gif)
