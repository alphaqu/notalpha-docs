---
tab_icon: "https://notalpha.dev/assets/dashloader/dashloader-transparent.png"
color: "#33ffbe"
---

# DashLoader documentation
To begin your journey with DashLoader, you need to add the notalpha.dev maven. You can do so by adding this to your build.gradle.
```groovy
repositories {
    maven {
        url 'https://notalpha.dev/maven/releases'
    }
}
```

And adding DashLoader as a **Compile time** dependency. This means DashLoader is actually not bundled with your mod and its not required to have DashLoader installed when running the game.
```groovy
dependencies {
    modCompileOnly 'dev.notalpha:DashLoader:5.0.0+1.19.3'
}
```