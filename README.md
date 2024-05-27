# Laboratorio de integracion de patrones de diseño

### Escenarios:

1. <strong>Global Configuration Management:</strong> Design a system that ensures a single, globally accessible configuration object without access conflicts.
### Solución:
> para este escenario se sugiere utilizar el patron singleton, ya que este restringe la creación de mas de una instancia  de clase, por lo que asegurara un unico objeto, adicionalmente se sugiere que para la implementacion de este singleton se aplique la doble  validación en la instanciacion del objeto para asegurar que en el acceso concurrente si 2 hilos intentan crear en el  mismo instante de tiempo una nueva instancia  no se permita mediante bloqueo de hilo y validacion de existencia de instancia.
### Simulación:
```

public class DatabaseConnectionSingleton {
   private static volatile Connection instance; // es el atributo que contendra la conexion 
   private static final Object lock = new Object(); // este atributo sera una bandera para determinar si el proceso de creacion de instancia esta en uso en otro hilo 

   private DatabaseConnectionSingleton() { // se hace el constructor privado para evitar la creacion mediante new y si se llama a nivel de misma clase se lanza excepcion para evitar su llamado 
       if (instance != null) {
           throw new RuntimeException("Utilizar getInstance para crear la conexion");
       }
   }

    // este metodo contendra la logica para instanciar la conexion es publico y estatico lo que permite su acceso de manera global
   public static Connection getInstance() {
       Connection result = instance; // se asigna en caso de que ya se contenga una conexion previa
       if (result == null) {  // se valida si ya se tiene conexion
           synchronized (lock) { // se aplica el bloqueo para uso seguro en hilos 
               result = instance;    // en caso de que en el inter antes del bloqueo ya haya occurido la instanciacion se asigna a result y se valida si ya se creo
               if (result == null) {   
                   try {
                        // aqui es donde ocurre la asignacion de la nueva conexion a base de datos
                       instance = result = DriverManager.getConnection("localhost", "usuario_demo", "password_demo");
                   } catch (SQLException e) {
                       e.printStackTrace();
                   }
               }
           }
       }
       return result; // se regresa la instancia de la conexión
   }
}

// en esta clase se ilustra  el uso para instanciar una conexion a bd mediante singleton
public class Main {
    public static void main(String[] args) {
       Connection connection = DatabaseConnectionSingleton.getInstance();
    }
}
```

2. <strong>Dynamic Object Creation Based on User Input:</strong> Implement a system to dynamically create various types of user interface elements based on user actions.
### Solución:
> Para este escenario el patron factory es el ideal ya que este permite definir una interfaz con un metodo,  el cual implementaran  las clases concretas  donde se definira la logica especifica de cada elemento de interfaz de usuario tal como el pintado de un  boton o un input, y para elegir que interfaz de usuario se pasara el nombre como parametro a el metodo de creacion.Tambien se puede utilizar un abstract factory en caso de que se requiera crear un factory de factories para escenarios donde existe una mayor variedad de componentes y subcomponentes
### Simulación:
```
// esta es la interfaz con el metodo que las clases concretas implementarán
public interface UIComponent {
    void render();
}

// esta clase concreta implementa la logica que realizara el pintado de la caja de texto
public class TextInput implements UIComponent {
    @Override
    public void render() {
        System.out.println("Caja de texto creada");
    }
}

// esta clase concreta implementa la logica que realizara el pintado del boton
public class Button implements UIComponent {
    @Override
    public void render() {
        System.out.println("Boton creado");
    }
}

// para claridad se define un enum con los tipos de interfaces de usuario
public enum UIComponentType {
    BUTTON,
    TEXT_INPUT
}

// esta clase define la logica para instanciar los elementos de interfaz de usuario segun el tipo de la enumeracion se instancia una u otra opcion o en caso de una opcion no valida se lanza una excepcion
public class UIComponentFactory {
    public UIComponent createUIComponent(UIComponentType type) { 
        switch (type) {
            case BUTTON:
                return new Button(); 
            case TEXT_INPUT:
                return new TextInput();
            default:
                throw new IllegalArgumentException("Opcion no válida");
        }
    }
}

// en esta clase se ilustra  el ejemplo  de como llamar al factory para generar un componente boton y una caja de texto
public class Main {
    public static void main(String[] args) {
        UIComponentFactory factory = new UIComponentFactory();
        UIComponent button = factory.createUIComponent(UIComponentType.BUTTON);
        UIComponent textInput = factory.createUIComponent(UIComponentType.TEXT_INPUT);
        button.render();
        textInput.render();
    }
}
```

3. <strong>State Change Notification Across System Components:</strong> Ensure components are notified about changes in the state of other parts without creating tight coupling.
### Solución:
> Para el escenario donde se requiere notificar de cambio en el estado de algun componente el patron observer permite subscribir alguna clase a un metodo que posee otro clase, con el fin de que cuando ocurra un cambio este notifique el cambio a la lista de componentes suscritos a esperar el cambio. Esta solucion es factible siempre y cuando el numero de subscriptores sea razonable ya que tiene la desventaja de que para numeros grandes pueda impactar el  performance
### Simulación:
```

// Se define la interfaz observador que la clase concreta implementara 
interface Observer {
    void update(String message);
}

// Esta es la clase concreta con la logica para el metodo de actualizacion que pintara cualquier mensaje de actualizacion que reciba, representa el observador de las actualizaciones
class User implements Observer {
    private String name;

    public User(String name){
        this.name = name;
    }

    @Override
    public void update(String message) {
        System.out.println("Usuario " + name + "  notificación: " + message);
    }
}

// Esta es la interfaz del subject que registrara los observadores de la lista de actualizacion
interface Subject {
    void registerObserver(Observer observer);
    void removeObserver(Observer observer);
    void notifyObservers(String message);
}

// Esta clase implementara la logica para agregar, eliminar, notificar  y generar las actualizaciones hacia la lista de observadores
class NotificationService implements Subject {
    private List<Observer> observers = new ArrayList<>();

    @Override
    public void registerObserver(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }

    // con este metodo se envia la notificación
    @Override
    public void notifyObservers(String message) {
        for (Observer observer: observers) {
            observer.update(message);
        }
    }
    
    // este metodo se encargará de generar un update que sera notificado a los observadores
    public void postMessage(String message) {
        System.out.println("Message Posted to all users: " + message);
        notifyObservers(message);
    }
}

//  En esta clase se lleva a cabo la creacion del servicio de notificacion, se crean 2 usuarios que se registran en la lista de observadores y 
public class Main {
    public static void main(String[] args) {
    
        NotificationService notificationService = new NotificationService(); // se crea el servicio de notificacion con la lista de observadores vacia
        
        // se crean 2 nuevos usuarios
        User user1 = new User("Jose");
        User user2 = new User("Miguel");
        
        // se agregan los 2 usuarios a la lista de observadores
        notificationService.registerObserver(user1); 
        notificationService.registerObserver(user2);
        
        // se genera un nuevo mensaje para que se ejecute la notificacion hacia los 2 usuarios registrados
        notificationService.postMessage("Actualización de precios realizada");
    }
}

```

4. <strong>Efficient Management of Asynchronous Operations:</strong> Manage multiple asynchronous operations like API calls which need to be coordinated without blocking the main application workflow.
### Solución:
> Para un escenario de operaciones asincronas el uso del patron promises representa una buena alternativa para el consumo de un API desde una pagina web, para java CompletableFuture permite llamada a algun api de manera asincrona sin bloquear el hilo
### Simulación:
```
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.util.concurrent.CompletableFuture;

public class Main {
    public static void main(String[] args) {
        // se genera un cliente para conexion via http
        HttpClient client = HttpClient.newHttpClient();
        
        // se genera un request para acceder como ejemplo a la pagina de facturapi
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://restful-api.dev/"))
                .build();

        // para el caso de java CompletableFuture es un ejemplo de promise
        // en este caso envia una solicitud asincrona
        CompletableFuture<HttpResponse<String>> response = client.sendAsync(request, HttpResponse.BodyHandlers.ofString());
        
        //mediante esta linea esperara en el body una respuesta, la cual al ser recibida sera impresa en la consola
        response.thenApply(HttpResponse::body).thenAccept(System.out::println).join();
        
        // Como muestra de que el flujo regular no es interrumpido al esperar por la respuesta del llamado al completable future, se puede imprimir mas informacion para mostrar el flujo asincrono
        System.out.println("Continuar con el flujo del hilo ...");
    }
}
```
