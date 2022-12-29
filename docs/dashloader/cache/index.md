---
icon: "bolt"
---
# Cache
Caching is how DashLoader manages to get its increadible speed when loading minecraft. A lot of minecrafts assets do not need to be computed but can instead instantly get loaded back by DashLoader. Your custom implementation of assets may be loaded by minecraft itself which is quite slow and adding caching support is very easy.