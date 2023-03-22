---
title: "DashObject"
icon: "data_array"
---
The `DashObject` is a serializable form of the Object you are trying to cache. 

So for example, `Identifier`s have a `DashIdentifier` which is a `DashObject`. This allows `Identifier`s to be added to the registry because the `DashIdentifier` provides a way to serialize and deserialize the `Identifier`.

???+ abstract

    ```java
	public class DashIdentifier implements DashObject<Identifier> /* (1)! */ {
		public final String namespace;
		public final String path;

	    //(2)
		public DashIdentifier(String namespace, String path) {
			this.namespace = namespace;
	        this.path = path;
		}

	    //(3)
		public DashIdentifier(Identifier identifier) {
			this.namespace = identifier.getNamespace();
			this.path = identifier.getPath();
		}

	    //(4)
		@Override
		public Identifier export(RegistryReader reader) {
			return new Identifier(namespace, path);
		}
	}
	```

	1. The DashObject interface is inherited by every DashObject - it includes methods like `export` which run on deserialization.
	2. This constructor contains every field so that Hyphen (the serializer we use) can create the object.
	3. This constructor gets run when a new Identifier tries to get cached. Your goal is to destructure the object so that we can serialize it. It is commonly referred as the "Factory" constructor.
	4. This export method gets run on deserialization. Here you recreate the original Identifier object.

??? tip "Real world examples"
	A simple [DashObject](https://github.com/alphaqu/DashLoader/blob/fabric-1.19/src/main/java/dev/notalpha/dashloader/client/identifier/DashIdentifier.java).

	A less simple [DashObject](https://github.com/alphaqu/DashLoader/blob/fabric-1.19/src/main/java/dev/notalpha/dashloader/client/model/DashBasicBakedModel.java).

	A way less simple [DashObject](https://github.com/alphaqu/DashLoader/blob/fabric-1.19/src/main/java/dev/notalpha/dashloader/client/shader/DashShader.java).

## Factory Constructor
When an Object gets added to a `RegistryWriter` it tries to find a DashObject that has support for that class. If it finds a DashObject, its "Factory" constructor will be called. 

The constructor can have 4 different signatures.

- FULL: Contains the target class and a `RegistryWriter` 
```java
public DashIdentifier(Identifier identifier, RegistryWriter writer);
```
- WRITER: Contains only a `RegistryWriter`
```java
public DashIdentifier(RegistryWriter writer); // kinda useless??
```
- RAW: Contains only the target class
```java
public DashIdentifier(Identifier identifier);
```
- EMPTY: Does not have any parameters.
```java
public DashIdentifier(); // where the party at?
```

So if we want to serialize an identifier we would add the fields for the namespace and path, and then extract them from the Identifier using the Factory constructor.
```java hl_lines="2-3 5-9"
public class DashIdentifier implements DashObject<Identifier> {
	public final String namespace;
	public final String path;

    // Factory constructor.
	public DashIdentifier(Identifier identifier) {
		this.namespace = identifier.getNamespace();
		this.path = identifier.getPath();
	}
}
```

Please note that this constructor must be public since otherwise DashLoader cannot access it.

## Serialization
DashLoader uses the Hyphen Serializer for maximum speed and efficiency. The documentation for that serializer is available on [https://notalpha.dev/docs/hyphen](/hyphen).

We will show a simple example here:

```java hl_lines="6-10"
public class DashIdentifier implements DashObject<Identifier> {
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
2. We have a constructor which takes in all the fields we defined. Please note that the arguments must have the same order as the fields, and it needs to be `#!java public`.


## Exporting
Exporting is when DashLoader has deserialized its cache and wants to now convert its DashObjects to the original Objects. The DashObject interface contains the methods DashLoader uses when exporting the objects.

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

So for our Identifier, our export method would look like this:

```java hl_lines="15-18"
public class DashIdentifier implements DashObject<Identifier> {
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
You might have noticed we have a RegistryWriter on the factory constructor and a RegistryReader on the exporting methods. You can read more about how to use the DashRegistry [here](../registry).