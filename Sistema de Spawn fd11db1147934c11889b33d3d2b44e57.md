# Sistema de Spawn

# Indice

- Introduction
- Codigos del sistema
- Detalles a tomar en cuenta
- Importancia de los Colisionadores
- Personalizacion
- Conclusión

# Introducción

El sistema de spawn es una funcionalidad muy importante en los juegos, ya que permite a los jugadores aparecer en un lugar específico al inicio del juego o después de morir. Asimismo puede ser usado para aparecer ciertos objetos y enemigos en el campo con los que el jugador puede interactuar.

Existen varios tipos de sistemas de spawn, como el spawn aleatorio, en el que se aparece en un lugar aleatorio del mapa, o el spawn fijo, donde se aparece siempre en el mismo lugar. También hay sistemas de spawn dinámicos, en los que el lugar de aparición cambia en función de ciertas condiciones del juego.

Para implementar un sistema de spawn, es necesario tener en cuenta varios factores. En primer lugar, es importante definir los puntos de spawn y determinar las condiciones para que se active cada uno. También es necesario tener en cuenta el sistema de colisión del juego, para evitar que los jugadores aparezcan en lugares inapropiados o dentro de objetos.

Además, el sistema de spawn debe ser lo suficientemente flexible como para permitir una fácil modificación y personalización por parte de los desarrolladores del juego. Esto incluye la posibilidad de agregar nuevos puntos de spawn, cambiar las condiciones de activación y modificar la lógica del sistema.

En resumen, un buen sistema de spawn es esencial para asegurar una experiencia de juego fluida y justa para los jugadores. Al implementar un sistema de spawn, es importante tener en cuenta los diferentes tipos de spawn y las condiciones necesarias para cada uno, así como la flexibilidad y personalización del sistema.

# Codigos del sistema

# Character controller

```csharp
public class CharacterController : MonoBehaviour
{

    public float VelMovimiento = 5.0f;
    public float jumpSpeed = 10.0f;

    private Rigidbody rb;
    private bool isGrounded = true;

    void Start()
    {
        rb = GetComponent<Rigidbody>();
    }

    void FixedUpdate()
    {
        float horizontalInput = Input.GetAxis("Horizontal");
        float verticalInput = Input.GetAxis("Vertical");

        Vector3 movement = new Vector3(horizontalInput, 0.0f, verticalInput);
        movement = movement.normalized * VelMovimiento * Time.deltaTime;

        if (movement.magnitude > 0)
        {
            transform.rotation = Quaternion.LookRotation(movement);//>Rota al jugador a la direccion a la que se mueve
        }

        rb.MovePosition(transform.position + movement);

        if (Input.GetButtonDown("Jump") && isGrounded)
        {
            rb.AddForce(Vector3.up * jumpSpeed, ForceMode.Impulse);
            isGrounded = false;
        }
    }

    void OnCollisionEnter(Collision collision)
    {
        if (collision.gameObject.CompareTag("Ground"))
        {
            isGrounded = true;
        }

        if (collision.gameObject.CompareTag("Enemy"))
        {
            Object.Destroy(gameObject);
        }
    }
}
```

# Camera Controller

```csharp
public class CameraController : MonoBehaviour
{
    public GameObject Player;
    private Vector3 offset;

    // Start is called before the first frame update
    void Start()
    {
        // Buscar el objeto del jugador
        Player = FindObjectOfType<CharacterController>().gameObject;
        offset = transform.position - Player.transform.position;
    }

    // Update is called once per frame
    void Update()
    {
        Player = FindObjectOfType<CharacterController>().gameObject;
        // Mover la cámara para seguir al jugador
        if (Player != null)
        {
            transform.position = Player.transform.position + offset;
        }
    }
}
```

# Boton Reaparición

```csharp
public class BotonReaparicion : MonoBehaviour
{
    public GameObject MediumMechStrikerMasterPrefab;
    public CameraController cameraController;
    // Start is called before the first frame update
    void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {
        
    }

    public void reaparecerjugador()
    {

        GameObject newPlayer = Instantiate(MediumMechStrikerMasterPrefab, new Vector3(-24, 1, -11), Quaternion.identity);
        cameraController.Player = newPlayer;
    }
}
```

Este codigo se encarga de reaparecer al objeto jugador de nuevo en la escena en caso de que los enemigos lo destruyan

# Enemy Spawner

```csharp
public class EnemySpawner : MonoBehaviour
{
    public GameObject EnemyPrefab;
    public float tiempotranscurrido = 0f;
    public float tiempoEntreEnemigos = 10f;
    public int maxEnemigos = 4; // Establece el número máximo de enemigos que pueden estar en el escenario.
    private int numEnemigos = 0; // Lleva el seguimiento del número actual de enemigos en el escenario.

    // Start is called before the first frame update
    void Start()
    {

    }

    // Update is called once per frame
    void Update()
    {
        tiempotranscurrido += Time.deltaTime;

        if (tiempotranscurrido > tiempoEntreEnemigos && numEnemigos < maxEnemigos) // Verifica si ha pasado suficiente tiempo y si hay espacio para un nuevo enemigo.
        {
            tiempotranscurrido = 0f;
            // Generar posición aleatoria dentro de un círculo de radio 1
            Vector3 posicionSpawn = Random.insideUnitCircle.normalized * 10f;
            //Esta linea crea un enemigo en posicon al azar
            Instantiate(EnemyPrefab, posicionSpawn, Quaternion.identity);
            numEnemigos++; // Incrementa el número de enemigos en el escenario.
            Debug.Log("Nuevo Enemigo en camino");

        }
    }
    void OnDestroy()
    {
        numEnemigos--; // Decrementa el número de enemigos en el escenario cuando un enemigo es destruido.
    }
}
```

Este codigo se encargar de aparecer enemigos en el escenario. Esta configurado para aparecer una instancia del prefab enemigo cada 3 segundos por default pero al ser una variable publica puede ser editado en el editor de unity. Asimismo se puede delimitar el numero maximo de enemigos permitidos en la escena, esto con el fin de que no aparezcan infinitamente.

# Spawn Manager

```csharp
public class SpawnManager : MonoBehaviour
{
    public GameObject objectToSpawn;
    // Update is called once per frame
    void Update()
    { 
        
    }
    // Este método se activa cuando el jugador entra en contacto con el objeto.
    void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            Vector3 randomSpawnPosition = new Vector3(Random.Range(-23, 23), 5, Random.Range(-20, 20));//
            Instantiate(objectToSpawn, randomSpawnPosition, Quaternion.identity);//Esta linea de codigo crea un nuevo objeto a partir del prefab que se use como base
            //Quaternion lo unico que hace es que el objeto que aparezca en la escena no tenga rotacion y spawnee como el prefab
        }
    }
}
```

Se encarga de administar la aparicion de objetos no animados al azar dentyro del mapa de juego. Esta delimitado de tal forma que los objetos aparezan al azar en X en un rango de (-23, 23), en Y en un valor fijo de 5 y en Z (-20, 20) tambien al azar. Si quisieran editar este rango solo basta con cambiar los numeros antes mencionados por los deseados.

# Game Manager

```csharp
public class GameManager : MonoBehaviour
{
    public static GameManager instance;
    private GameObject player;
    public int EnemigosTotales { get { return enemigosTotales; } }
    private int enemigosTotales = 0;

    private void Awake()
    {
        //Este codigo se asegura que solo haya una instancia de gamemanager
        if (instance != null && instance != this)
        {
            Destroy(this.gameObject);
        }
        else
        {
            instance = this;
        }

        // Obtener la referencia al objeto del jugador
        player = GameObject.FindGameObjectWithTag("Player");
    }

    public GameObject GetPlayer()
    {
        return player;
    }

    public void SumarEnemigos(int enemigosASumar)
    {
        if (enemigosASumar > 0)
        {
            enemigosTotales += enemigosASumar;
            Debug.LogFormat("Enemigos totales: {0}", enemigosTotales);
        }
    }
}
```

Este codigo se encarga de administar las variables que otros codigos necesitan para funcionar correctamente. En su mayoria se encarga de dar una referencia universal del objeto Player para todos los scripts que lo requieran.

# Detalles a tomar en cuenta

Es importante tomar en cuenta la eficiencia del sistema de spawn para evitar retrasos en la carga de la escena y para que sea compatible con diferentes plataformas. Además, es recomendable usar sistemas de spawn que permitan la personalización de los puntos de spawn y las condiciones de activación, así como el uso de un sistema de colisión adecuado para evitar problemas de aparición en lugares inapropiados. 

# Importancia de los Colisionadores

Los colisionadores son una parte esencial del sistema de spawn, ya que permiten evitar que los jugadores aparezcan en lugares inapropiados o dentro de objetos. Es importante asegurarse de que los colisionadores estén configurados correctamente y sean lo suficientemente precisos para evitar problemas de colisión. Además, es recomendable utilizar un sistema de colisión dinámico que pueda detectar cambios en el entorno de juego y ajustar los puntos de spawn en consecuencia. Asismismo es necesario tenerlos configurados correctamente ya que de no estarlo los scripts anexados a los objetos con colisionadores podrian no funcionar adecuadamente. Si deseas hacer alguna modificacion a algun componente de la escena es importante que mantengas la configuracion de colisionador lo más limpia posible para evitar errores.

# Personalización

La personalización del sistema de spawn es esencial para adaptarlo a las necesidades específicas de cada juego. Esto incluye la capacidad de agregar nuevos puntos de spawn, cambiar las condiciones de activación y modificar la lógica del sistema en función de las necesidades del juego. Además, es importante que el sistema de spawn sea lo suficientemente flexible como para permitir una fácil integración con otros sistemas del juego, como el sistema de colisión y el sistema de inteligencia artificial de los enemigos.

Este sistema tiene muchas variables de caracter publico para que la mayoria de las funciones que se quieran personalizar puedan hacerse desde el editor de unity en orden para no interfererir demasiado con el codigo evitando problemas adicionales. De ser necesario editar el codigo, las partes clave de este vienen comentadas para que alguien inexperto sepa que hace cada linea de codigo en el editor de unity.

# Conclusión

En resumen, el sistema de spawn es un componente vital en cualquier juego. Para implementarlo adecuadamente, es necesario tener en cuenta los diferentes tipos de spawn y las condiciones necesarias para cada uno, así como la flexibilidad y personalización del sistema. También es importante prestar atención al sistema de colisión del juego y asegurarse de que los colisionadores estén configurados correctamente. Al seguir estos consejos, se puede crear un sistema de spawn eficiente y personalizable que mejore la experiencia de juego para los jugadores.