---
icon: "app_registration"
title: "Adding DashObjects"
---
Once you have made a DashObject, DashLoader needs to be able to find it. The DashObject classes are designed to be easily removed incase DashLoader is behind on a Minecraft release.

```json title="fabric.mod.json"
{
    "custom": {
		"dashloader:dashobject" : [
			"you.yourmod.compat.dashloader.DashMyThingie"
		]
	},
}
```