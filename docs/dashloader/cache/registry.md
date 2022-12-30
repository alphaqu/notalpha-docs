---
icon: "database"
---
# Registry
The DashLoader Registry is a data object which holds all the DashObjects in a cache. Its job is to provide an easy way to deduplicate and export a huge amount of data really fast. The Registry is split into two classes, RegistryWriter is active on cache creation and RegistryReader is used on cache reading.

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

	1. The `model` field is a registry identifier and points to our child model.
	2. Here we use the `#!java int RegistryWriter.add(V)` method to add the child model to the registry and retrieve its registry ID which we will save.
	3. Here we get our child model back by using `#!java RegistryReader.get(int)` which will retrieve the object on deserialization.

## RegistryWriter
The RegistryWriter is a registry creator and its job is to track dependencies and create an exporting sequence which makes the exporting parallel while preventing race conditions.

The RegistryWriter has one single method.
```java 
public <R> int add(R object);
```
When calling add, you provide an object which has a DashObject implementor to the Registry, the registry saves the object and returns an ID which you can use to retrieve the object on the next cache load.

Internally it works by first checking if the object already exists in the deduplication map, then it tries to see if any DashObject directly supports the object. If that also fails it will seek the `MissingHandler`s which are fallback implementations for unknown objects.

## RegistryReader
The RegistryReader is the cache registry provider. Its job is to export all the DashObjects into their original forms and allow parents to get their children back.

```java 
public <R> R get(int id);
```
This method gets the original non-dash object that you initially added by using `#!java int add(R object)`. 
The order that RegistryReader exports ensures that the object you are getting has already been exported.

## ID
A registry ID is a 32-bit number which contains a 6-bit chunk number and a 26-bit chunk-entry number. You can retrieve the values by using the methods in `RegistryUtil`

This also means that DashLoader has a limit on 63 DashObject chunks and a limit on 67108863 for DashObject entries. If you ever hit this limit please let me know - either the values might get adjusted or we switch to using a long.