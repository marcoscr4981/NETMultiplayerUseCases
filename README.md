# Multiplayer Use Cases

He realizado las modificaciones en la escena **RPCs** donde la cápsula del jugador muestra un mensaje. He añadido la opción de que también **cambie automáticamente de color**.

Para ello he modificado el script `MoodManager.cs`.

## Propiedades

He añadido las siguientes propiedades:

- **NetworkVariable<Color32> m_NetworkedColor:** Variable de red que almacena el color del jugador y se sincroniza automáticamente entre el servidor y todos los clientes.
- **Material m_Material:** Propiedad donde se almacena la referencia al material del GameObject para poder modificar su color.

```csharp
NetworkVariable<Color32> m_NetworkedColor = new NetworkVariable<Color32>();
Material m_Material;
```

## Método `Awake()`

Al instanciar esta clase, se guarda la referencia al material del GameObject.

```csharp
void Awake()
{
    m_Material = GetComponent<Renderer>().material;
}
```

## Método `OnNetworkSpawn()`

Este método se ejecuta cuando el objeto aparece en la red. Aquí se realiza la suscripción al evento _OnValueChanged_ de la _NetworkVariable_, que se ejecutará automáticamente cuando el valor del color cambie en la red.

```csharp
public override void OnNetworkSpawn()
{
    base.OnNetworkSpawn();
    
    // Aplicar el valor actual del color cuando el objeto aparece
    OnClientColorChanged(m_Material.color, m_NetworkedColor.Value);

    // Nos suscribimos al evento que se ejecuta cuando cambia el valor de la NetworkVariable
    m_NetworkedColor.OnValueChanged += OnClientColorChanged;
}
```

## Método `OnNetworkDespawn()`

Cuando el objeto desaparece de la red se elimina la suscripción al evento _OnValueChanged_ para evitar referencias innecesarias.

```csharp
public override void OnNetworkDespawn()
{
    base.OnNetworkDespawn();
    
    // Nos desuscribimos del evento OnValueChanged
    m_NetworkedColor.OnValueChanged -= OnClientColorChanged;
}
```

## Método `ServerMoodMessageReceivedRpc`

Este método es un **Server RPC**, por lo que se ejecuta en el servidor cuando un cliente envía un mensaje. Aquí se genera un color aleatorio y se guarda en la _NetworkVariable_, lo que provoca que el nuevo color se sincronice automáticamente en todos los clientes.

```csharp
[Rpc(SendTo.Server)]
void ServerMoodMessageReceivedRpc(string message)
{
    // Resto de código

    // Obtenemos un color aleatorio de la clase MultiplayerUseCasesUtilities
    // y lo guardamos en la NetworkVariable para sincronizarlo en todos los clientes
    m_NetworkedColor.Value = MultiplayerUseCasesUtilities.GetRandomColor();
}
```

## Método `OnClientColorChanged()`

Este método se ejecuta automáticamente en los clientes cuando cambia el valor de la **NetworkVariable m_NetworkedColor**. Recibe el color anterior y el nuevo color, y se encarga de actualizar el material del jugador.

```csharp
void OnClientColorChanged(Color32 previousColor, Color32 newColor)
{
    m_Material.color = newColor;
}
```