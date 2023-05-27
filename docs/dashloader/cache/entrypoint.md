---
icon: "start"
---
???+ abstract
    Here we register our entrypoint to fabric.mod.json

    ```json title="fabric.mod.json"
    	"entrypoints": {
    		"dashloader": [
                "you.yourmod.compat.dashloader.DashLoaderCompatibility"
    		]
    	},
    ```
    
    This is our implementation of the Entrypoint
    ```java title="DashLoaderCompatibility.java"
    public class DashLoaderCompatibility implements DashEntrypoint/* (1)! */{
        /* (2)! */
	    @Override
    	public void onDashLoaderInit(CacheFactory factory) {
           factory.addDashObject(DashMyObject.class)/* (3)! */;
           factory.addModule(new MyModule())/* (4)! */;

		   /* (5)! */
		   factory.addMissingHandler(MyGoofyObject.class, (goofyObject, registryWriter) -> {
	    		if (goofyObject instanceof LessGoofyableObject m) {
	    			return new DashMyObject(m.innerCoolness);
	    		} else {
	    			return null;
	    		}
	    	})
    	}	
    }
    ```

	1. We implement DashEntrypoint to get access to the methods
	2. This method gets run once every time DashLoader initializes (once every session).
	3. Here we register our custom DashObject to DashLoader.
    4. Here we can register a custom module to DashLoader.
    5. We can add a custom MissingHandler to the DashRegistry which is a fallback function if the registry cannot find a DashObject.

??? tip "Real world example"
    [DashLoaderClient](https://github.com/alphaqu/DashLoader/blob/fabric-1.19.4/src/main/java/dev/notalpha/dashloader/client/DashLoaderClient.java)

DashLoader has a Fabric API Entrypoint for registering DashObjects, Modules and Missing Handlers.

You simply create a class which implements `DashEntrypoint` and add it to your fabric.mod.json under the `dashloader` key
```json title="fabric.mod.json"
	"entrypoints": {
		"dashloader": [
          "you.yourmod.compat.dashloader.DashLoaderCompatibility"
		]
	},
```

```java title="DashLoaderCompatibility.java"
public class DashLoaderCompatibility implements DashEntrypoint {

	@Override
	public void onDashLoaderInit(CacheFactory factory) {
	}
}
```


## Adding DashObjects
After you have made your DashObject you need to tell DashLoader about it. You do this by calling `addDashObject` in `CacheFactory`

```java title="DashLoaderCompatibility.java" hl_lines="3"
	@Override
	public void onDashLoaderInit(CacheFactory factory) {
        factory.addDashObject(DashMyObject.class);
    }
```

## Adding Modules
If you want to add a custom module, you need to tell DashLoader about it by calling `addModule` in `CacheFactory`
```java title="DashLoaderCompatibility.java" hl_lines="3"
	@Override
	public void onDashLoaderInit(CacheFactory factory) {
        factory.addModule(new MyModule());
    }
```

## MissingHandlers
When a DashLoader registry fails to find a DashObject for a given Object it will resort to its last option, `MissingHandler`s.

These are functions which check if the object is one of a fallbackable kind and map it to a bruhDashObject else it will return `#!java null` and let the other MissingHandlers find a use for it, if the registry runs out of MissingHandlers before its found a non-null returning handler, it will throw an exception saying that it was unable to add the object to the registry.

```java hl_lines="3-12"
	@Override
	public void onDashLoaderInit(CacheFactory factory) {
		factory.addMissingHandler(
			MyGoofyObject.class/* (1)! */,
			(goofyObject, registryWriter) -> {
				if (goofyObject instanceof LessGoofyableObject m) {
					return new DashMyObject(m.innerCoolness)/* (2)! */;
				} else {
					return null/* (3)! */;
				}
			}
		);
    }
```

1. This is the class the registry will be matching towards. If it finds that the object is of that class or one of its sub types it will call this function in hopes of getting a DashObject out of it.
2. We have found a LessGoofyableObject! Which in this example has the same fields as MyObject, so we create a DashMyObject and everything is awesome!
3. Returning null lets the Registry go on trying to find a way to serialize the object another way.