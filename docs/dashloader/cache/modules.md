---
icon: "apps"
title: "Modules"
---
A DashLoader Module is a caching implementation. It gets added on DashLoader's EntryPoint and is responsible for saving and loading data into/from the cache. 
???+ abstract
    We add our Module to DashLoader.
    ```java title="DashLoaderCompatibility.java"
    public class DashLoaderCompatibility implements DashEntrypoint {
		@Override
		public void onDashLoaderInit(CacheFactory factory) {
			factory.addModule(new MyModule()/* (1)! */);
		}	
	}
    ```

	1. We add our `DashModule` implementation to the DashCache.

	This is our epic implementation
	```java title="MyModule.java"
	public class MyModule implements DashModule {	
		// (1)	
		public static final class Data {
			public final List<Integer> myObjects;

			public Data(List<Integer> myObjects) {
				this.myObjects = myObjects;
			}
		}

		// (2) 
		@Override
		public Class<Data> getDataClass() {
			return Data.class;
		}

		// (3)
		@Override
		public Data save(RegistryWriter writer, StepTask task) {
			List<Integer> myObjects = new ArrayList<>();

	
			for (MyObject object : MY_OBJECTS) {
				myObjects.add(writer.add(object)/* (5)! */);
			}

			return new Data(myObjects/* (6)! */);
		}

		// (4)
		@Override
		public void load(Data mappings, RegistryReader reader, StepTask task) {
			for (Integer object : mappings.myObjects) {
				MY_OBJECTS.add(reader.get(object)/* (7)! */);
			}
		}
	}
	```


	1. This is our Data class that will get saved in the cache. Note that the class, fields and constructor needs to be public for our serializer to access it.
	2. We tell DashLoader what our DataClass is
	3. This method runs every time DashLoader is creating a cache, in here we create our data object.
	4. This method runs every time DashLoader is loading back its cache, in here we use our data object.
	5. In this example we add our objects to the registry and put our registry ids into a list.
	6. We save the list of registry ids in our data object.
	7. We go through our registry ids in our data object and get the original object back through the registry.

??? tip "Real world example"
    [ModelModule](https://github.com/alphaqu/DashLoader/blob/fabric-1.19.4/src/main/java/dev/notalpha/dashloader/client/model/ModelModule.java)


## Registration
To add a module to the DashLoader, you will need to call `#!java CacheFactory.addModule(DashModule)` in your DashEntrypoint.

```java title="DashLoaderCompatibility.java" hl_lines="5"
public class DashLoaderCompatibility implements DashEntrypoint {

	@Override
	public void onDashLoaderInit(CacheFactory factory) {
		factory.addModule(new MyModule());
	}	
}
```
```java title="MyModule.java"
public class MyModule implements DashModule {
	@Override
	public Data save(RegistryWriter writer, StepTask task) {
		// ...
	}

	@Override
	public void load(Data mappings, RegistryReader reader, StepTask task) {
		// ...
	}

	@Override
	public Class<Data> getDataClass() {
		// ...
	}
}
```

## Data
Every `DashModule` holds a Data class which is a serializable structure that DashLoader saves.

```java title="MyModule.java"
	@Override
	public Class<Data> getDataClass() {
		return Data.class;
	}

	// (1)	
	public static final class Data {
		// (2)	
		public final List<Integer> myObjects;

		// (3)
		public Data(List<Integer> myObjects) {
			this.myObjects = myObjects;
		}
	}
```

1. You need to make this class public as the Serializer would not be able to access the class.
2. All of the non-transient fields need to be public for the Serializer to access them.
3. You need to have a public constructor which contains all of the non-transient fields.

???+ warning
	It is not recommended to save big amounts of data into the Data object directly, this is because it is single threaded and does not fragment which means that there is a size limit on the total mappings cache. 
	
	It is instead recommended to use the DashRegistry system. You get a `#!java RegistryWriter` which consumes the object by `add()` and returns an `#!java int id`. 
	This is later used in `DashModule.load()` to get the original object back by calling `#!java RegistryReader.get(int id)`.

	Read more about the registry system [here](../registry)



## Saving
The `save` method runs when DashLoader is creating a cache. 
The purpose of the save method is to create the Data object and store useful information for the cache load.

```java title="MyModule.java"
	@Override
	public Data save(RegistryWriter writer, StepTask task) {
		List<Integer> myObjects = new ArrayList<>();

	
		for (MyObject object : MY_OBJECTS/* (1)! */) {
			myObjects.add(writer.add(object)/* (2)! */);
		}

		/* (3)! */
		return new Data(myObjects);
	}
```

1. Here we iterate through our objects that we want to cache.
2. We add the objects to the RegistryWriter, and it returns an `#!java int id` which we can use on cache load to get our object back. Please note that the object needs to have a [DashObject](../dashobject) implementation behind it, else the registry will have no way to serialize it.
3. Return the data object you specified in `getDataClass()`

After this, DashLoader does it's magic on its own to store both the registry objects and the mappings' data objects.

## Loading
The `load` method runs when DashLoader is loading back its cache.

The Data class will be deserialized together with the Registry, now you will need to put your cached objects to work!
```java title="MyModule.java"
	@Override
	public void load(Data mappings, RegistryReader reader, StepTask task) {
		for (Integer object : mappings.myObjects/* (1)! */) {
			MY_OBJECTS.add(reader.get(object)/* (2)! */);
		}
	}
```

1. We go through the list of registry ids which we saved on the last game load in our module Data object.
2. We get the object back from the registry by our id and save it back into our list.

And now we have successfully cached our objects! Note that you will need to disable your normal computation of objects if DashLoader is present. 
Else what is the point of caching things if you still compute them.

## Other
### Task weight
If your caching a lot of data you may want to increase the amount of weight your step takes in the progress bar. The default weight is 100 but may be increased to whatever you like.
```java
	@Override
	public float taskWeight() {
		return 69420f;
	}
```

### Optional Modules
Modules may be deactivated at runtime. This disables the module from running `save` and `load`.

You can control if the module is active by implementing this method.
```java
	@Override
	public boolean isActive() {
		return MyMod.checkIfModuleStuffShouldBeActiveOrNot();
	}
```
### CachingData
CachingData is a data class which holds intermediary minecraft data for loading/saving.
This is less useful for third party support as you need to use these where the data is normally loaded. Which means that you now need to depend on DashLoader as a runtime dependency (not good).
But for DashLoader they are really useful and are how DashLoader keeps its code from looking like garbage.



#### Adding the data
When DashLoader switches its state (saving or loading) it will call the `reset` method in the Modules. This is used by `CachingData` to reset its inner state to an idle object. 
Please note the default object **cannot be null**.

```java title="MyModule.java"
	// (1)
	public static final CachingData<List<MyObject>> MY_CACHING_OBJECTS = new CachingData<>();

	// (2)
	@Override
	public void reset(Cache cacheManager) {
		MY_CACHING_OBJECTS.reset(cacheManager, new ArrayList<>()/* (3)! */);
	}
```

1. We create a public static field which holds our intermediary data.
2. This is called on save/load to reset the inner data object. Else you might accidentally keep data across reloads and have duplicate data.
3. This is the default value of the `CachingData`. Please note it cannot be null.

???+ warning
	The `reset` method will also run on `Cache.Status.IDLE`. So if you don't have a check for it, DashLoader will clear the data after the game is loaded.

	This is why CachingData is not that useful for external support but is great for DashLoader which caches the Minecraft data and needs to later discard the intermediary data.


#### Caching
Then we can use this in our module save/load.

```java hl_lines="5 18"
	@Override
	public Data save(RegistryWriter writer, StepTask task) {
		List<Integer> myObjects = new ArrayList<>();
	
		for (MyObject object : MY_CACHING_OBJECTS.get(Cache.Status.SAVE/* (1)! */)) {
			myObjects.add(writer.add(object));
		}

		return new Data(myObjects);
	}

	@Override
	public void load(Data mappings, RegistryReader reader, StepTask task) {
		List<Integer> myObjects = new ArrayList<>();
		for (Integer object : mappings.myObjects) {
			myObjects.add(reader.get(object));
		}
		MY_CACHING_OBJECTS.set(Cache.Status.LOAD/* (2)! */, myObjects);
	}
```

1. We specify to the CachingData what stage DashLoader is required to be in for the method to function. If the state is wrong it will throw an exception. This is to prevent bugs where saving code gets run accidentally on loading in mixin code.
2. We specify to the CachingData what stage DashLoader is required to be in for the method to function. If the state is wrong it will throw an exception. This is to prevent bugs where saving code gets run accidentally on loading in mixin code.

#### Using the data
Here we can `visit` the data when the cache status is matched. 

Here the lambda will only get run when the cache status is SAVE. 
In this case we are putting our computed objects into the data for caching.
```java
MyModule.MY_CACHING_OBJECTS.visit(Cache.Status.SAVE, map -> {
	map.putAll(myObjectList);
});
```

Here we get our potentially cached objects back.
```java
MyModule.MY_CACHING_OBJECTS.visit(Cache.Status.LOAD, cachedObjects -> {
	myObjectList.putAll(cachedObjects);
});
```
