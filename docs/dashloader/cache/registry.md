---
icon: "database"
---
# Registry
The Registry is the core of DashLoader. This is a data structure which contains all of the data DashLoader saves, you use it by providing an Object with a [DashObject](../dashobject) implementation to the registry and then using the returned id to save its registry location for next load. Then on the next game launch, you can use that id to get your original object back.

The reason why we don't save objects directly is because the Registry does dependency tracking and multithread's as hard as possible when exporting the objects back into their original form.

???+ abstract
	```java
	public class DashParentModel {
		public final int model; // (1)!

		public DashParentModel(int model) {
			this.model = model;
		}

		public DashParentModel(ParentModel parent, RegistryWriter writer) {
			this.model = writer.add(parent.getChild()); // (2)!
		}

		public ParentModel export(RegistryReader reader) {
			return new ParentModel(reader.get(model)/* (3)! */);
		}
	}
	```

	1. The `#!java int model` field is a registry ID which points to our child model.
	2. Here we use the `#!java int RegistryWriter.add(V)` method to add the child model to the registry and retrieve its registry ID which we will save.
	3. Here we get our child model back by using `#!java RegistryReader.get(int)` which will retrieve the object from the registry.

??? tip "Real world example"	
	[In the `image` field, we are saving a NativeImage](https://github.com/alphaqu/DashLoader/blob/fabric-1.19/src/main/java/dev/notalpha/dashloader/client/font/DashBitmapFont.java)

??? note "Source code"	
	[RegistryWriter](https://github.com/alphaqu/DashLoader/blob/fabric-1.19.4/src/main/java/dev/notalpha/dashloader/api/registry/RegistryWriter.java)
	<- [Implementation](https://github.com/alphaqu/DashLoader/blob/fabric-1.19.4/src/main/java/dev/notalpha/dashloader/registry/RegistryWriterImpl.java)
	& [Dependency tracking implementation](https://github.com/alphaqu/DashLoader/blob/fabric-1.19.4/src/main/java/dev/notalpha/dashloader/registry/TrackingRegistryWriterImpl.java)


	[RegistryReader](https://github.com/alphaqu/DashLoader/blob/fabric-1.19.4/src/main/java/dev/notalpha/dashloader/api/registry/RegistryReader.java)
	<- [Implementation](https://github.com/alphaqu/DashLoader/blob/fabric-1.19.4/src/main/java/dev/notalpha/dashloader/registry/RegistryReaderImpl.java)



## RegistryWriter
The RegistryWriter is a registry creator. Its job is to track dependencies and create an exporting sequence which makes the exporting parallel while preventing race conditions.

The RegistryWriter has one single method.
```java 
public <R> int add(R object);
```
When calling add, you provide an object to the Registry, then it saves the object and returns an ID which you can use to retrieve the object on the next cache load.

!!! note
	A DashObject implementation which supports the object needs to exist for the registry to serialize/deserialize it, else the registry has no idea what to do with it and throws an exception.

#### Implementation Details
It works by first checking if the object already exists in a deduplication map, then it tries to see if any DashObject directly supports the object. If that also fails it will seek the `MissingHandler`s which are fallback implementations for unknown objects that *hopefully* find a DashObject for that object, else it will throw an exception.

The RegistryWriter also tracks dependencies if it's invoked from a DashObject, this allows it to build a dependency tree which allows objects to be loaded in parallel while ensuring that all of the children have already been deserialized.

## RegistryReader
The RegistryReader is used on cache load to retrieve objects from the registry.

```java 
public <R> R get(int id);
```
You call `get` with the id that you got by using `#!java <R> int add(R object)` to get the cached object back from the registry.

#### Implementation Details
On the cache save the RegistryWriter took note of what you registered and build a dependency tree which ensured that all of your registered objects are exported before you are.

This method is also insanely fast, its basically just a two dimensional array lookup.

## ID
A registry ID is a 32-bit number which contains a 6-bit chunk number and a 26-bit chunk-entry number. You can retrieve the values by using the methods in `RegistryUtil`

This also means that DashLoader has a limit of 63 DashObject chunks and a limit of 67108863 DashObject entries. If you ever hit this limit please let me know - the values might get adjusted or we switch to using a long.