---
icon: "data_array"
---
# DashObject
The DashObject is a serializable form of the Object you are trying to cache. 

So for example an Identifier has a DashIdentifier which is a DashObject and is the object that dashloader serializes into the cache file. Where it later loads it back and exports the DashIdentifier to Identifier.

???+ abstract

    ```java
	@DashObject(Identifier.class) /* (1)! */
	public class DashIdentifier implements Dashable<Identifier> /* (2)! */ {
		public final String namespace;
		public final String path;

	    //(3)
		public DashIdentifier(String namespace, String path) {
			this.namespace = namespace;
	        this.path = path;
		}

	    //(4)
		public DashIdentifier(Identifier identifier) {
			this.namespace = identifier.getNamespace();
			this.path = identifier.getPath();
		}

	    //(5)
		@Override
		public Identifier export(RegistryReader reader) {
			return new Identifier(namespace, path);
		}
	}
	```

	1. This is the DashObject annotation which says that we are trying to add caching support to Identifier
	2. The Dashable interface is inherited by every DashObject, it includes methods like `export` which run on deserialization.
	3. This constructor contains every field so that Hyphen (the serializer we use) can create the object.
	4. This constructor gets run when a new Identifier tries to get cached. Your goal is to destructure the object so that we can serialize it. It is commonly refered as the "Factory" constructor.
	5. This export method gets run on deserialization. Here you recreate the original Identifier object.

## Metadata
DashLoader needs to understand what exactly it is handling and what the DashObject is trying to do. This is why the `DashObject` annotation exists, it tells DashLoader what we are trying to add caching support to. Every DashObject which is defined in `fabric.mod.json` needs to have this annotation.

In this example we are adding support to `Identifier` by making a `DashIdentifier` have the annotation `DashObject` with the *target* `Identifier.class`
```java
@DashObject(Identifier.class)
public class DashIdentifier {}
```

## Factory Constructor
When an Object gets added to a `RegistryWriter` it tries to find a DashObject that has support for that class. If it finds a DashObject, it's "Factory" constructor will be called. 

The constructor can have 4 different signatures.

- FULL: Contains the target class and a `RegistryWriter` 
```java
public DashIdentifier(Identifier identifier, RegistryWriter writer);
```
- WRITER: Contains only a `RegistryWriter`
```java
public DashIdentifier(RegistryWriter writer);
```
- RAW: Contains only the target class
```java
public DashIdentifier(Identifier identifier);
```
- EMPTY: Does not have any parameters.
```java
public DashIdentifier(); // why tho?
```

So if we want to serialize an identifier we would add the fields for the namespace and path, and then extract them from the Identifier using the Factory constructor.
```java hl_lines="3-4 6-10"
@DashObject(Identifier.class)
public class DashIdentifier {
	public final String namespace;
	public final String path;

    // Factory constructor.
	public DashIdentifier(Identifier identifier) {
		this.namespace = identifier.getNamespace();
		this.path = identifier.getPath();
	}
}
```

Please note this constructor must be public else DashLoader cannot access it.

## Serialization
DashLoader uses the Hyphen Serializer because of it's speed and efficiency. The documatation for that serializer is available on [docs.notalpha.dev/hyphen](docs.notalpha.dev/hyphen).

But we will show a simple example here.

```java hl_lines="7-11"
@DashObject(Identifier.class)
public class DashIdentifier {
    // (1)
	public final String namespace;
	public final String path;
        
    // (2)
	public DashIdentifier(String namespace, String path) {
		this.namespace = namespace;
        this.path = path;
	}

	public DashIdentifier(Identifier identifier) {
		this.namespace = identifier.getNamespace();
		this.path = identifier.getPath();
	}
}
```

1. The fields that need to be serialized need to be `#!java public`. If you want a temporary field, mark it as `#!java transient`
2. We have a constructor which takes in all of the fields we defined. Please note the arguments must have the same order as the fields and it needs to be `#!java public`.


## Exporting
Exporting is when DashLoader has deserialized its cache and wants to now convert its DashObjects to the original Objects. The Dashable interface is inherited by all DashObjects and it contains the methods DashLoader uses when exporting the objects.

```java
// The preExport methods runs before the DashObject is exported. 
// Please note this method runs on the main thread and is not multithreaded.
void preExport(RegistryReader reader);

// The export method exports the original object from the DashObject. 
// This method is heavily multithreaded.
Identifier export(RegistryReader reader);

// The postExport methods runs after the DashObject is exported. 
// Please note this method runs on the main thread and is not multithreaded.
void postExport(RegistryReader reader);
```

So for our Identifier, an example of export method looks like this.

```java hl_lines="2 16-19"
@DashObject(Identifier.class)
public class DashIdentifier implements Dashable<Identifier> {
    public final String namespace;
    public final String path;

    public DashIdentifier(String namespace, String path) {
        this.namespace = namespace;
        this.path = path;
    }

    public DashIdentifier(Identifier identifier) {
        this.namespace = identifier.getNamespace();
        this.path = identifier.getPath();
    }

	@Override
	public Identifier export(RegistryReader reader) {
		return new Identifier(namespace, path);
	}
}
```


## Registry
You might have noticed we have a RegistryWriter on the factory constructor and a RegistryReader on the exporting methods. You can read more about how to use the DashRegistry here.